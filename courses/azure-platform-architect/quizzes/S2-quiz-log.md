# S2 quiz log

## 2.1 Microsoft Entra ID Native Integration (2026-11-28)

Score: **6/7** (6/6 on 2.1, plus one Section 1 re-test). Placeholder-only format instructions.

| # | Topic | Answer | Correct | Result |
|---|---|---|---|---|
| 1 | Blocked at sign-in, nothing in `system.access.audit` → Entra sign-in logs | D | D | ✓ |
| 2 | CA on the AzureDatabricks app covers browser + Azure CLI; Databricks OAuth secret bypasses Entra (TWO) | CD | CD | ✓ |
| 3 | Order: P1 + CA Administrator → new policy → target AzureDatabricks app → grant control and enable | BACD | BACD | ✓ |
| 4 | Partner from another tenant → B2B guest in the home tenant, AIM uses email | B | B | ✓ |
| 5 | Sign in again every 4 hours → CA sign-in frequency | A | A | ✓ |
| 6 | Managed services CMK → AzureDatabricks app gets Key Vault Crypto Service Encryption User | C | C | ✓ |
| 7 | Re-test (S1 1.1): what an Owner can do with the managed RG (TWO) | AB | BD | ✗ |

### Weak spots

- **Managed RG deny assignment: missed again** (S1 mixed quiz Q1, now 2.1 Q7). Both times picked "add an NSG rule". The **system deny assignment blocks writes to every resource in the managed RG, Owner included**, so NSG rules, disk resizes and VM changes are all out. What's left: **read** (browse, view the deny assignment under IAM), **delete the whole workspace** (takes the RG with it), and **choose the RG name at creation**. If you need your own NSG rules, that's **VNet injection**, where the VNet and NSG live in your own RG. Re-test in the next quiz.
- 2.1 content: none this round.

## 2.2 Automatic Identity Management (2026-11-28)

Score: **7/7** (6/6 on 2.2, plus one Section 1 re-test). Placeholder-only format instructions.

| # | Topic | Answer | Correct | Result |
|---|---|---|---|---|
| 1 | Service principal appears on first use, not when its group is added | A | A | ✓ |
| 2 | AIM: any Entra edition, no admin role; nested groups and SPs (TWO) | BD | BD | ✓ |
| 3 | Order: readiness report → enable AIM → validate in audit log → disable SCIM | CADB | CADB | ✓ |
| 4 | Entra deletion deactivates but doesn't revoke tokens → Token report | C | C | ✓ |
| 5 | Synced groups, including child groups, need workspace assignment | D | D | ✓ |
| 6 | AIM off: users and SPs stay, AIM groups lose members → set up SCIM first | B | B | ✓ |
| 7 | Re-test (S1 1.1): deny assignment blocks Owner's NSG change; VNet injection to own the NSG (TWO) | AC | AC | ✓ |

### Weak spots

- **Managed RG deny assignment: fixed** on this re-test (missed in the S1 mixed quiz and the 2.1 quiz). Re-test once more in a later section to confirm it has stuck.
- 2.2 content: none this round.
