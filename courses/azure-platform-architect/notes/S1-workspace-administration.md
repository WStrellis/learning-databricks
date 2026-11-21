# S1: Azure Databricks Workspace Administration Fundamentals

Course: [Azure Databricks Platform Architect](https://customer-academy.databricks.com/learn/courses/7541/azure-databricks-platform-architect) (Databricks Academy)

Overlap with DP-750: the basics of workspaces, tiers, and control vs compute plane are in [LP1-M1 Explore Azure Databricks](../../dp-750/notes/LP1-M1-explore-azure-databricks.md). The notes below cover what this course adds or puts differently.

---

## 1.1 Azure Databricks Service Architecture

Lecture, 12 min. Theme: *where Databricks runs on Azure, and who owns what.*

Objectives: describe the first-party service model and the control/compute plane split; explain the managed resource group and its deny assignment.

### A. A first-party Azure service ⚠️

- Azure Databricks is a **Microsoft first-party service**. Resource provider: **`Microsoft.Databricks`**. Workspaces are Azure resources in **your** subscription.
- Every DBU lands on your **Azure invoice as Azure consumption**. No separate Databricks bill, no Marketplace SaaS subscription to set up.
- Because it's Azure consumption, it **can count toward an Azure consumption commitment (MACC)**, but check the eligible-services list for your agreement in **Cost Management + Billing** instead of assuming.

**Who owns what:**

| Piece | Lives in | What's there |
|---|---|---|
| **Control plane** | Databricks-managed Azure subscription, **in your workspace region** | Web app, REST APIs, cluster manager, notebooks, jobs, Unity Catalog |
| **Serverless compute plane** | Databricks-managed subscription, same region as the workspace | Serverless SQL, jobs, notebooks, serverless workspaces. You pay DBUs that **include the machine cost** and configure nothing in Azure Compute. |
| **Classic compute plane** | **Your subscription**, inside the **managed resource group** | Cluster VMs, disks, NICs. The control plane launches them on your behalf. |
| Your data | Your subscription | Your ADLS Gen2 accounts. Serverless reaches them with **Unity Catalog credentials**. |

- Users and tools reach the control plane over **HTTPS 443** and authenticate with **Microsoft Entra ID** out of the box.
- Classic workspaces show as **"Hybrid"** in the Azure portal: compute is in your subscription, control plane is not.
- The Azure Databricks service has a **fixed Entra application ID**: `2ff814a6-3304-4ab8-85cb-cd0e6f879c1d`. It's the **token audience** when you get an Entra token for Databricks (more in Section 2).
- **What's on the invoice:** DBU meters, plus (for classic) VM, disk, network, and storage meters.

### B. The managed resource group ⚠️

A **classic** workspace creates a second resource group that you name but **don't control**. Convention: `databricks-rg-{workspace}-{unique}`.

| What lives there | The deny assignment | What you can still do |
|---|---|---|
| Workspace storage account (**`dbstorage...`**): system data, job output, DBFS root, workspace catalog | Named *"System deny assignment created by Azure Databricks"*. Created and owned by Azure, not you. | **Read** everything: browse the group, view resources, list the deny assignment under **Access control (IAM)** |
| Managed VNet (**`workers-vnet`**, default prefix **10.139**) and its NSG, unless you used VNet injection | Denies the listed actions for the listed principals. Inspect its permissions, scope, and exclusions. | **Delete the whole workspace**, which deletes the group with it |
| **NAT gateway** when secure cluster connectivity (SCC) uses the managed VNet (**billed**) | A matching deny **beats an Owner role assignment**. System-protected deny assignments can't be edited or deleted by the customer. | **Choose the group name** at creation and reference it in ARM, Bicep, Terraform |
| **Access connector** with a managed identity for the storage account | | |
| Cluster VMs, managed disks, NICs, IPs while clusters run | | |

**Name it once.** The managed RG name is **fixed at creation**.
- Bicep usually derives it as `databricks-rg-${workspaceName}-${uniqueString(...)}`.
- Terraform exposes it as **`managed_resource_group_name`**, and changing it **recreates the workspace**.
- Pick a naming convention that fits your CMDB and cost reports.

> Exam angle: "Subscription Owner can't delete a VM / change the NSG in the managed RG. Why?" → the **deny assignment** overrides RBAC allows, Owner included. The fix is not a role change.

### C. Workspace URL anatomy and the account console

| Element | Form | Notes |
|---|---|---|
| **Per-workspace URL** | `https://adb-{workspace-id}.{random}.azuredatabricks.net` | Workspace ID = the number between `adb-` and the first dot. Same ID shows in the account console, after `?o=` on legacy URLs, and in Terraform outputs. |
| Legacy regional URL | `https://{region}.azuredatabricks.net/?o={workspace-id}` | **Avoid.** May not work with new workspaces, slower, and breaks features that need the workspace ID in the host. |
| **Account console** | `https://accounts.azuredatabricks.net` | **One Databricks account per Microsoft Entra tenant.** Account ID is under your username menu, not shown inside a workspace. |
| Azure resource ID | `/subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.Databricks/workspaces/{name}` | What Azure Policy, RBAC, diagnostic settings, and the Azure CLI use |

**Portal vs account console** (which knob lives where):

| Azure portal | Account console |
|---|---|
| Creates and configures the **resource**: tier, networking, encryption, diagnostics, tags | Governs the **Databricks estate**: users and groups, metastores, workspace assignments, network connectivity (NCCs), usage |

### D. Ways into the platform

All commercial entry points are Azure entry points, since it's first party.

| Path | Key facts |
|---|---|
| **Azure portal** (default) | Create a resource → Analytics → Azure Databricks. Pay-as-you-go on the Azure bill from the first DBU. **Needs a subscription that isn't a Free Trial.** |
| **14-day trial tier** | Pricing tier *"Trial (Premium - 14-Days Free DBUs)"* chosen at workspace creation. **Free DBUs only**: VM, disk, storage, and network meters still bill. **Classic compute only**: the wizard greys out the **Serverless workspace type** and disables **customer-managed keys, infrastructure encryption, and the Enhanced Security and Compliance add-on**. Upgrade to Premium when the 14 days end. |
| **Pre-purchase plan** | Buy **Databricks commit units (DBCU)** for **1 or 3 years** under **Azure Reservations**. Applies across workloads and tiers, deducted automatically. **All purchases final**: no exchange or cancellation. |
| **Databricks Order with an Azure Commit** | Committed-use agreement negotiated **directly with Databricks** that counts Azure Databricks usage. Usage still flows through Azure. Confirm the transaction path with your account team. |
| **Free Edition** | No cost, **serverless-only**, quota-limited, hosted by Databricks. **Replaced Community Edition in 2025.** Not in your Azure subscription. For learning and hobby use, not production. |

> Exam trap: the trial is "free" for **DBUs only**. Infrastructure still bills. And trial workspaces can't use CMK or serverless workspaces.

### Key takeaways

- First-party Azure service (`Microsoft.Databricks`), billed as Azure consumption.
- Control plane runs in the Databricks subscription. Classic compute runs in a managed RG in **yours**. Serverless compute runs in **theirs**.
- The managed RG is protected by a **system deny assignment**. Inspect the denied actions and excluded principals in IAM.
- Workspace URL `adb-{id}.{random}.azuredatabricks.net`. Account console `accounts.azuredatabricks.net`, one account per Entra tenant.
- Entry points: portal PAYG, 14-day Premium trial, DBCU pre-purchase, Databricks Order with an Azure Commit, Free Edition.

### Course vs docs

- No conflicts found with the DP-750 notes or current docs. The course matches them on: Standard tier gone for new workspaces, Free Edition replacing Community Edition, and serverless compute in the Databricks-managed subscription.
- Typo on the course page: the "Databricks Order with an Azure Commit" card says *"only ADI Services bill directly"*. It's not clear what "ADI" means. Ask the account team or check before relying on it.

### Official references (from the lesson, checked 2026-10-27)

- [Introduction](https://learn.microsoft.com/en-us/azure/databricks/introduction/)
- [Secure cluster connectivity](https://learn.microsoft.com/en-us/azure/databricks/security/network/classic/secure-cluster-connectivity)
- [Workspace details (URLs, IDs)](https://learn.microsoft.com/en-us/azure/databricks/workspace/workspace-details)
- [Admin concepts](https://learn.microsoft.com/en-us/azure/databricks/admin/admin-concepts)
- [Free trial](https://learn.microsoft.com/en-us/azure/databricks/getting-started/free-trial)
- [Azure deny assignments](https://learn.microsoft.com/en-us/azure/role-based-access-control/deny-assignments)
- [Azure Databricks pricing](https://azure.microsoft.com/en-us/pricing/details/databricks/)
- [Databricks reservations (DBCU)](https://learn.microsoft.com/en-us/azure/cost-management-billing/reservations/reservation-discount-databricks)
- [Storage firewall support](https://learn.microsoft.com/en-us/azure/databricks/security/network/storage/firewall-support)

### ⚡ Quick review

- [ ] Where do the control plane, classic compute, and serverless compute each run?
- [ ] What's in the managed RG, and why can't an Owner change it?
- [ ] What 3 things *can* you still do with the managed RG?
- [ ] Can you change the managed RG name later? What does Terraform do if you try?
- [ ] How do you find the workspace ID from the URL? Why avoid the regional URL?
- [ ] How many Databricks accounts per Entra tenant?
- [ ] What does the 14-day trial not cover, and what features does it disable?
- [ ] DBCU pre-purchase: terms, scope, refundable?

---

<!-- Next: 1.2 Workspace Types and Tiers -->
