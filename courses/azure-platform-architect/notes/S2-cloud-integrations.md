# S2: Azure Databricks Cloud Integrations

Course: [Azure Databricks Platform Architect](https://customer-academy.databricks.com/learn/courses/7541/azure-databricks-platform-architect) (Databricks Academy)

Overlap with S1: tenant binding, the first account admin, and the account console Security page are in [S1 1.4 Account Administration on Azure](S1-workspace-administration.md#14-account-administration-on-azure). The Entra application ID first came up in [S1 1.1](S1-workspace-administration.md#11-azure-databricks-service-architecture). The notes below cover what this section adds.

---

## 2.1 Microsoft Entra ID Native Integration

Lecture, 10 min. Theme: *identity you don't have to wire.*

Objectives: explain why Azure Databricks needs no SSO configuration; apply Conditional Access and tenant MFA implications.

### A. No SSO to configure ⚠️

- On **AWS and Google Cloud**, an admin sets up SSO in the account console. On **Azure there's nothing to configure.**
- **Entra ID login is on by default** for the account console and every workspace, over **OpenID Connect**. The account is bound to your tenant.
- The account console still has a **Security → Identity provider setup** tab. On Azure, the work there is **identity management and federation policies**, not choosing a sign-in method.

**A browser login to a workspace** (lesson diagram):

```mermaid
sequenceDiagram
    participant U as User (browser)
    participant W as Workspace<br/>adb-{id}.{n}.azuredatabricks.net
    participant E as Microsoft Entra ID<br/>(tenant)
    participant CA as Conditional Access<br/>(policies for AzureDatabricks app)
    U->>W: 1. Open workspace URL
    W->>E: 2. OpenID Connect redirect (audience 2ff814a6-...)
    E->>CA: 3. Evaluate policies for app AzureDatabricks
    CA-->>E: 4. Require MFA / compliant device / allow
    E-->>U: 5. Sign-in and MFA challenge
    U->>W: 6. ID token (tid, oid, upn)
    W-->>U: 7. Session; JIT provisioning if first login
```

**The details that matter:**

| Item | Fact |
|---|---|
| **The application** | Enterprise app **`AzureDatabricks`** in every tenant, fixed app ID **`2ff814a6-3304-4ab8-85cb-cd0e6f879c1d`** |
| What that app ID is used for | **Audience of Entra ID access tokens** issued for Azure Databricks, **target of Conditional Access policies**, and the principal you grant **Key Vault Crypto Service Encryption User** (or an access policy with get, wrapKey, unwrapKey) for **customer-managed keys** |
| **Tenant binding** | Account tied to the tenant ID. Moving a workspace to another tenant is **unsupported**: it stays in the original account. Several tenants → the account console opens your **default** tenant's account; reach another tenant's account from a workspace in that tenant. |
| **Users** | A workspace admin can add **any user in the workspace's Entra tenant, B2B guests included**. Automatic identity management uses the **guest's email as the username**. **Identity federation is on for every new workspace and can't be turned off.** |
| **Token claims checked** | **`aud`** = the app ID, **`iss`** = `https://sts.windows.net/{tenant-id}/`, **`tid`** = the workspace's tenant |

> Exam angles:
> - "Configure SSO for Azure Databricks" → nothing to configure. Entra ID over OIDC is native.
> - "Which object do you target with Conditional Access / grant Key Vault access for CMK?" → the **AzureDatabricks** enterprise app, ID `2ff814a6-...`.
> - "Entra ID token rejected" → check `aud`, `iss`, `tid` against the app ID and the workspace's tenant. (Databricks OAuth tokens and federated tokens use other audiences; see Course vs docs.)

### B. Conditional Access and MFA inherit from the tenant ⚠️

| Where policy lives | What you can require | Side effects to expect |
|---|---|---|
| **Entra admin center → Entra ID → Conditional Access → New policy**. Target resource: search the app ID or the name **AzureDatabricks**. | **MFA**, **compliant or hybrid-joined devices**, **named locations**, **sign-in risk** levels | A CA policy on **Azure DevOps** plus one on Azure Databricks can **break Git folder clones** with a denied-access error. Fix: exclude Azure Databricks IPs or the affected users. |
| Needs **Entra ID P1**. Risk-based policies need **ID Protection (a P2 feature)**. Role to create them: **Conditional Access Administrator**. | **Session controls** such as **sign-in frequency** | **Service principals** are governed by **Conditional Access for workload identities**, a separately licensed feature (**Microsoft Entra Workload ID Premium**). |
| The Azure security baseline lists CA for the data plane as **supported and enabled by default**. | Policies run **whenever Entra issues the token**: browser logins **and** Entra token flows like the **Azure CLI**. A **Databricks OAuth secret never passes through Entra**, so **no CA policy touches it**. | **Blocked sign-ins never reach Databricks.** The evidence is in the **Entra sign-in logs**, not the Databricks audit table. |

> Exam angles:
> - "User blocked at login, nothing in `system.access.audit`. Where do you look?" → **Entra sign-in logs**.
> - "Require MFA for a service principal" → **not possible**: service principals can't do MFA. CA for **workload identities** (Workload ID Premium) can only **block** access, based on location or risk. And a **Databricks OAuth secret** (M2M) bypasses Entra entirely, so CA never sees it.
> - "Git folder clone suddenly fails with access denied" → CA on Azure DevOps interacting with Databricks. Exclude Databricks IPs or the users.
> - "Licensing for risk-based CA" → P2 (ID Protection). Basic CA → P1.

### C. Guests and cross-tenant scenarios ⚠️

| Scenario | What works |
|---|---|
| Partner user from another tenant | Invite them as a **B2B guest in the workspace's home tenant**, then add them. AIM uses the guest's **email** as the username. |
| Users spread across several tenants | AIM reads the **home tenant only**, including matching B2B guest objects there. It **doesn't read another tenant's directory**. Check guest identity matching and the **migration readiness report**. |
| Publishing to Power BI as a guest | **Not supported.** Users publish only to Power BI workspaces in their **home** tenant. |
| Workspace moved to a new tenant | **Unsupported.** The workspace stays in the original account. Plan a **new workspace and migrate**. |

### D. What stays configurable on the Databricks side

| Provisioning | Authentication controls | Perimeter |
|---|---|---|
| **Automatic identity management (AIM)** toggle and its **account access denylist (up to 100 rules)**, both under **Security → Identity provider setup → Identity management** | **Federation policies** for OAuth token federation: **account-wide** under Identity provider setup, **per service principal** under its **Credentials & secrets** tab | **IP access lists** for the account console and for workspaces, **1,000 entries each** |
| **Just-in-time (JIT) provisioning**: on by default for accounts created after **May 1, 2025**, and **always on when AIM is enabled** | **Token report** under Security: filter, export, and **revoke personal access tokens account-wide** | **Context-based ingress control** is the recommended primary control (**GA for workspace policies**, **Beta for the account-console policy**) |
| **SCIM** where you still need it (lesson 2.2) | Workspace toggles for **PATs and their maximum lifetime** (lesson 2.3) | **Private Link** and public network access (Section 3) |
| | | **No Databricks-side session timeout is documented.** Sign-in frequency is an **Entra Conditional Access session control**. |

**Don't carry AWS vocabulary over.** *Unified login* and *emergency access* are documented for AWS and Google Cloud accounts only. The Azure docs cover neither, even though the generic description of the Identity provider setup tab lists a Unified login section. The Azure version of "emergency access" is a **break-glass Entra account excluded from Conditional Access** that holds **Contributor or Owner on the workspace resource**: that user gets **workspace admin** when they click **Launch Workspace** in the Azure portal (same rule as [S1 1.4](S1-workspace-administration.md#14-account-administration-on-azure)).

> Exam angles:
> - "Make users sign in again every 4 hours" → **Entra CA sign-in frequency**, not a Databricks setting. Note: sign-in frequency is a **periodic reauthentication interval**, not an inactivity timeout. It's enforced when the app next goes to Entra for a token, so it doesn't guarantee an idle session ends at the mark.
> - "Revoke every PAT a departed user created, across workspaces" → **account console Token report**.
> - "Stop AIM from bringing in certain users" → the AIM **account access denylist** (max 100 rules).
> - "Emergency access if CA locks everyone out" → break-glass Entra account excluded from CA, with Contributor/Owner on the workspace.

### Key takeaways

- Entra ID login is native: OIDC against the **AzureDatabricks** app (`2ff814a6-3304-4ab8-85cb-cd0e6f879c1d`). No identity provider to configure in the account console.
- Conditional Access and MFA are **tenant policies** (Entra ID P1) targeting that app. Blocked sign-ins show only in **Entra** logs.
- AIM supports matching B2B guest objects in the **home tenant**. It can't read another tenant's directory.
- Workspaces can't move between tenants.
- Databricks-side controls: AIM and JIT, federation policies, token report, IP access lists, context-based ingress. **Periodic reauthentication (sign-in frequency) belongs to Entra.**

### Course vs docs

- **Two dates, two features. Don't mix them up.** This lesson: **JIT provisioning** is on by default for accounts created after **May 1, 2025**. [S1 1.4](S1-workspace-administration.md#14-account-administration-on-azure): **AIM** is on for accounts created after **August 1, 2025**. Not a conflict, but an easy swap on a quiz.
- "No Databricks-side session timeout is documented" is the course's wording for Azure. The Entra answer is sign-in frequency, which forces **periodic reauthentication**. It isn't an idle timeout ([session lifetime docs](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-session-lifetime)). Recheck the Databricks side before relying on it, since workspace settings change often.
- **Token audience.** The lesson says the app ID is the audience of *every* token. That's true for **Entra ID access tokens**, but [OAuth federation policies](https://learn.microsoft.com/en-us/azure/databricks/dev-tools/auth/oauth-federation-policy) accept the audiences you configure (default: the Databricks account ID), and Databricks-issued OAuth tokens don't come from Entra at all.
- **Service principals and MFA.** Service principals can't do MFA. [CA for workload identities](https://learn.microsoft.com/en-us/entra/identity/conditional-access/workload-identity) supports only the **Block** grant control.
- Things likely to change: context-based ingress for the account console (Beta), and the AIM denylist limit (100 rules).
- No conflicts found with S1 or DP-750 notes.

### Official references (from the lesson, checked 2026-10-27)

- [Authentication and access control](https://learn.microsoft.com/en-us/azure/databricks/security/auth/)
- [Manage users](https://learn.microsoft.com/en-us/azure/databricks/admin/users-groups/users)
- [Get Entra ID tokens manually](https://learn.microsoft.com/en-us/azure/databricks/dev-tools/auth/aad-token-manual)
- [API access permissions](https://learn.microsoft.com/en-us/azure/databricks/security/auth/api-access-permissions)
- [Workload identities FAQ](https://learn.microsoft.com/en-us/entra/workload-id/workload-identities-faqs)
- [Automatic identity management](https://learn.microsoft.com/en-us/azure/databricks/admin/users-groups/automatic-identity-management/)
- [Migrate to AIM](https://learn.microsoft.com/en-us/azure/databricks/admin/users-groups/automatic-identity-management/migrate-to-aim)
- [SCIM provisioning with Entra ID](https://learn.microsoft.com/en-us/azure/databricks/admin/users-groups/scim/aad)

### ⚡ Quick review

- [ ] Why is there no SSO to configure on Azure? What protocol does the login use?
- [ ] What's the AzureDatabricks app ID, and what 3 things is it used for?
- [ ] Which 3 token claims does a workspace check, and what must each equal?
- [ ] What Entra license does Conditional Access need? Risk-based policies?
- [ ] How do you put Conditional Access on a service principal? Does CA apply to a Databricks OAuth secret?
- [ ] A user is blocked at sign-in. Where's the evidence?
- [ ] Why might Git folder clones break, and how do you fix it?
- [ ] How do you bring in a partner from another tenant? Can AIM read their home tenant?
- [ ] Can a B2B guest publish to Power BI in your tenant?
- [ ] Where do you force periodic reauthentication? Is that the same as an idle timeout?
- [ ] JIT default date vs AIM default date
- [ ] What's the Azure equivalent of AWS "emergency access"?

---

<!-- Next: 2.2 Automatic Identity Management -->
