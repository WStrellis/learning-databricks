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

## 1.2 Workspace Types and Tiers

Lecture, 10 min. Theme: *Serverless or Hybrid, and Premium either way.*

Objectives: choose between classic and serverless workspace types; treat Premium as the only tier going forward.

Two decisions frame every deployment: the **workspace type** (chosen once at creation) and the **pricing tier** (no longer a real choice).

### A. Workspace types ⚠️

- Portal **Workspace type** selector: **Serverless** or **Hybrid** (classic).
- In ARM it's **`computeMode`**: **required at creation and never changeable**.
- A serverless workspace comes preconfigured with serverless compute and **default storage**. It needs **no VNet, no storage account, and no managed resource group** in your subscription.

**Decision flow** (any "yes" → Classic/Hybrid, with VNet injection recommended):

1. **Need compute inside your own VNet?** Traffic inspection, ExpressRoute, back-end Private Link.
2. **Classic-only workload?** RDD code, **Scala or R** as the main language, time-based streaming triggers, Lakeflow Connect to on-premises or private databases.
3. **Region or preview gap?** Check the feature-region table for serverless workspaces.
4. All "no" → **Serverless workspace**: quick start, no cloud wiring.

**Currency check:** serverless workspaces went **GA on Azure on January 30, 2026**. They're in most commercial regions that have serverless compute (Australia East, East US 2, West Europe, UK South, and more). **Four regions have serverless compute but no serverless workspaces: Canada East, China North 3, Jio India West, UK West.** Check the region table before promising one to a customer.

### B. What isn't created for a serverless workspace ⚠️

| Workspace property (ARM) | Classic (Hybrid) | Serverless |
|---|---|---|
| `managedResourceGroupId` | **Required** | Not allowed |
| `parameters` (VNet injection, subnets, NAT gateway, storage account, `enableNoPublicIp`) | Available | Not allowed |
| `accessConnector`, `defaultStorageFirewall`, `defaultCatalog`, `requiredNsgRules` | Available | Not allowed |
| `encryption.entities.managedDisk`, infrastructure (double) encryption | Available | Not allowed |
| `encryption.entities.managedServices`, `enhancedSecurityCompliance` | Available | **Supported** |
| `publicNetworkAccess` | Front-end **and** back-end Private Link | **Front-end-only** Private Link |

**Default storage:**
- Fully managed object storage **in the Databricks account**. It backs internal storage, workspace storage, and the workspace's default catalog.
- Catalogs on default storage can be created **only from serverless workspaces**.
- Dropped objects have a **7-day recovery window** by default, then purge within 48 hours.

**Default storage constraints:**
- **Classic compute can't read data in default storage.** Other workspaces can be granted access, but they must use serverless compute.
- **No credential vending or direct file access** for external Iceberg and Delta clients. BI tools work over ODBC/JDBC, and the Files API serves volumes.
- ODBC/JDBC results larger than about **5 MiB (Cloud Fetch) fail over front-end Private Link.**

**Mixed estates:**
- Keep shared data in **your own ADLS Gen2**, through external locations or catalog managed locations.
- A serverless workspace can still connect to your storage accounts for managed and external tables.
- Rule of thumb: **default storage for the workspace catalog and quick starts, your storage for the lake.**

### C. Premium is the only tier ⚠️

**Standard tier end of life:**
- **April 1, 2026:** new workspaces must be **Premium**.
- **October 1, 2026:** automatic upgrades of the remaining Standard workspaces to Premium **began**, rolling out in phases.
- The portal, ARM (`sku.name`), and Terraform (`sku`) still list `standard` and `trial` values, so **expect old examples in the wild.**

**What Premium always gated:**
- Unity Catalog and metastore attachment
- Databricks SQL, serverless compute, Lakeflow pipelines, Mosaic AI, Lakebase
- RBAC (access control), IP access lists, Private Link, diagnostic logs
- Customer-managed keys, double encryption, the Enhanced Security and Compliance add-on

**Upgrading a Standard workspace:**
- Azure portal: open the workspace → **Upgrade to Premium**. ARM or API also work.
- **VNet-injected workspaces upgrade only through the portal or an ARM template.**
- Notebooks, users, and cluster configurations are kept. **Running clusters may terminate.**
- **Access control settings start disabled after the upgrade, and can't be disabled once enabled.**

**Where to see the tier:** the **account console → Workspaces** page shows each workspace's current tier. It's the quickest way to inventory an estate during the phased upgrade.

### D. Serverless vs classic side by side

| Dimension | Serverless workspace | Classic (Hybrid) workspace |
|---|---|---|
| Azure prerequisites | **None beyond a resource group** | Managed RG name, optional VNet and subnets, NSG, NAT gateway or firewall path |
| Compute location | Databricks subscription, same region | Your subscription (managed RG), plus optional serverless |
| Workspace storage | **Default storage** (Databricks-managed) | The `dbstorage` account in the managed RG |
| Network control | Egress network policies, **NCC private endpoints**, front-end Private Link | VNet injection, UDRs, Azure Firewall, front-end and back-end Private Link |
| Encryption scopes | **Managed services key only** | Managed services, managed disks, workspace storage, double encryption |
| Who can create it | Still an Azure resource: **Contributor or Owner** at subscription level (or the documented custom role). No VNet, storage, or networking rights needed. | The same Azure role, **plus Network Contributor on the VNet** if you inject one |

> Exam angles:
> - "Must use Scala/R, ExpressRoute, or inspect traffic" → **Hybrid**.
> - "Wants CMK for managed disks, or back-end Private Link" → **Hybrid**. Serverless supports only the managed services key and front-end Private Link.
> - "Classic cluster can't read a table in default storage" → expected. Move shared data to your own ADLS Gen2.
> - "Change an existing workspace from Hybrid to Serverless" → **not possible**. `computeMode` is fixed. Create a new workspace.

### Key takeaways

- Workspace type is Serverless or Hybrid (classic), set by `computeMode` at creation and never changed.
- Serverless workspaces need no managed RG, VNet, storage account, or access connector. Only the managed services key and the compliance add-on apply.
- Classic compute can't read default storage. Keep shared data in your own ADLS Gen2.
- Serverless workspaces are GA on Azure (January 30, 2026). Check the region table.
- No new Standard workspaces from April 1, 2026. Phased automatic upgrades to Premium began October 1, 2026.

### Course vs docs

- No conflicts with the DP-750 notes: both say Standard is gone for new workspaces from April 1, 2026. This lesson adds the **October 1, 2026 automatic upgrade** of existing Standard workspaces.
- The course itself warns that one portal deployment page still shows a **"Public Preview"** label for serverless workspaces even though they're GA. If an exam question or doc page calls them preview, it's out of date.

### Official references (from the lesson, checked 2026-10-27)

- [Serverless workspaces](https://learn.microsoft.com/en-us/azure/databricks/admin/workspace/serverless-workspaces)
- [Default storage](https://learn.microsoft.com/en-us/azure/databricks/storage/default-storage)
- [Standard tier retirement](https://learn.microsoft.com/en-us/azure/databricks/admin/account-settings/standard-tier)
- [Account settings](https://learn.microsoft.com/en-us/azure/databricks/admin/account-settings/account)
- [Feature region support](https://learn.microsoft.com/en-us/azure/databricks/resources/feature-region-support)
- [Create a metastore](https://learn.microsoft.com/en-us/azure/databricks/data-governance/unity-catalog/create-metastore)
- [Enable workspaces for Unity Catalog](https://learn.microsoft.com/en-us/azure/databricks/data-governance/unity-catalog/enable-workspaces)
- [Release notes, January 2026](https://learn.microsoft.com/en-us/azure/databricks/release-notes/product/2026/january)

### ⚡ Quick review

- [ ] What ARM property sets the workspace type? Can you change it later?
- [ ] Name 3 reasons you'd still need a Hybrid workspace.
- [ ] Which 4 regions have serverless compute but no serverless workspaces?
- [ ] Which encryption and Private Link options does a serverless workspace support?
- [ ] Can classic compute read default storage? Where should shared data live?
- [ ] Key dates: Standard tier end for new workspaces, and the start of automatic upgrades
- [ ] What's kept and what can break when you upgrade Standard → Premium? What about VNet-injected workspaces?
- [ ] Extra Azure role needed to create a VNet-injected workspace?

---

<!-- Next: 1.3 Deploying Workspaces -->
