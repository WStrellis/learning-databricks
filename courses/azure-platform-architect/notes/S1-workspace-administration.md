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
- The "Databricks Order with an Azure Commit" card says *"only ADI Services bill directly"*. This isn't a typo: lesson 1.4 links "ADI Services" to [docs.databricks.com/adi](https://docs.databricks.com/adi) and says it's an integration setting for eligible accounts, covered in lesson 2.5. Fill in the details then.

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

## 1.3 Deploying Workspaces

Lecture, 12 min. Theme: *the ARM resource and the ways to write it.*

Objectives: deploy through the portal, ARM/Bicep, and Terraform; identify the creation-time decisions that can't be changed later.

Every deployment tool ends in the same place: one `Microsoft.Databricks/workspaces` resource with a handful of properties. DP-750 covered the four basic creation methods (portal, IaC, PowerShell, CLI) in [LP1-M1 §2](../../dp-750/notes/LP1-M1-explore-azure-databricks.md#2-workspaces-creating-one). This lesson goes deeper into the properties and into what's fixed for life.

### A. Prerequisites ⚠️

**Azure RBAC**
- **Contributor or Owner at subscription level**, or a custom role with `Microsoft.Databricks/workspaces/*`, `Microsoft.Databricks/accessConnectors/*`, resource group read/write, and deployment read/write/validate.
- **VNet injection adds Network Contributor on the VNet**, or `subnets/join/action` and `subnets/write`.
- **The creator becomes the first workspace admin, and stays admin even if the Azure role is removed later.**

**Resource providers and quotas**
- Register **`Microsoft.Databricks`**, plus `Microsoft.Compute`, `Microsoft.ManagedIdentity`, `Microsoft.Storage`, and `Microsoft.Network` in the subscription.
- **vCPU quota per VM family** in the region.
- **Free Trial subscriptions can't create workspaces at all.**
- Limits worth knowing: 980 resource groups per subscription, and 250 storage accounts per region per subscription (1,000 on request).

**Region and feature availability**
- 45 Azure regions. Some are served by a control plane in a paired region.
- Serverless workspaces, serverless compute, and Managed DR each have their own region lists.
- **Azure Government regions lack the account console, Databricks SQL, and Unity Catalog.**

**Deployment states:** Provisioning, Running, Failed, Banned, Cancelling. Creation usually takes minutes but can take hours. **A failed deployment still leaves a workspace resource in the Failed state. Delete it and redeploy instead of trying to repair it.**

### B. The workspace resource ⚠️

Resource type `Microsoft.Databricks/workspaces`. **Latest stable API: `2026-01-01`**, the first stable version with the required `computeMode` property (`2024-05-01` doesn't have it).

| Property | Purpose |
|---|---|
| `sku.name` | `premium`, or `trial` for the 14-day Premium trial. The schema still accepts `standard`, but new Standard workspaces have been refused since April 1, 2026. |
| `computeMode` | `Hybrid` (classic) or `Serverless`. **Required on create, immutable.** |
| `managedResourceGroupId` | Full resource ID of the managed RG. Required for Hybrid, not allowed for Serverless. |
| `parameters.customVirtualNetworkId`, `customPublicSubnetName`, `customPrivateSubnetName` | VNet injection: your VNet, plus the host and container subnets |
| `parameters.enableNoPublicIp` | **Secure cluster connectivity (SCC).** Defaults to true from API `2024-05-01` and is becoming mandatory. |
| `parameters.natGatewayName`, `publicIpName`, `vnetAddressPrefix`, `storageAccountName`, `storageAccountSkuName` | Managed VNet and workspace storage naming and sizing |
| `parameters.prepareEncryption`, `requireInfrastructureEncryption` | Managed identity for the workspace storage CMK; double encryption (**creation only**) |
| `publicNetworkAccess`, `requiredNsgRules` | Private Link posture. `Disabled` + `NoAzureDatabricksRules` for back-end Private Link (Section 3). |
| `encryption.entities.managedServices` and `managedDisk` | Customer-managed keys for control-plane data and cluster disks |
| `defaultStorageFirewall`, `accessConnector` | Workspace storage firewall, with an access connector outside the managed RG |
| `enhancedSecurityCompliance` | Compliance security profile, enhanced security monitoring, automatic cluster update |
| `defaultCatalog` | Initial name and type of the workspace's default catalog |

### C. Portal, Bicep, and the IaC alternatives

| Tool | Key facts |
|---|---|
| **Azure portal** | Create a resource → Analytics → Azure Databricks. **Basics** tab: name, region, tier, **Workspace type**. **Networking** tab: SCC (No Public IP, defaults to Yes) and the VNet injection switch. Setting public network access to No means no public network access. Then the Encryption, Security and compliance, and Tags tabs → Review + create. Good for learning the moving parts, not for your tenth workspace. |
| **ARM / Bicep** | The native declaration. `az deployment group create --template-file main.bicep` with a `.bicepparam` file. **`what-if` previews the change.** The Azure Verified Module **`avm/res/databricks/workspace`** wraps the resource. Access connectors, role assignments, VNets, and diagnostics live in the same template. |
| **Terraform / OpenTofu** | `azurerm_databricks_workspace` (azurerm 5.x) with a `custom_parameters` block; `azurerm_databricks_access_connector` for identity. ⚠️ **The resource has no `computeMode` argument yet, so a serverless workspace needs Bicep or the `azapi` provider.** The `databricks` provider (1.140 at the time of writing) then configures the account and the workspace. OpenTofu runs the same providers from its own registry (`tofu init / plan / apply`). |
| **stackql-deploy** | A manifest plus `.iql` query files, run by `stackql-deploy build {stack-dir} {stack-env}`. Workspaces and access connectors are `azure_isv.databricks.workspaces` and `access_connectors`. Account-level objects come from the `databricks_account` provider. |
| **Pulumi** | `azure_native.databricks.Workspace` and `AccessConnector` in Python, TypeScript, Go, or .NET. The `pulumi_databricks` provider mirrors the Terraform resources for the account and workspace layers. State and secrets in Pulumi Cloud or your own backend. |
| **Azure CLI** | `az databricks workspace create` with `--vnet`, `--public-subnet`, `--private-subnet`, `--enable-no-public-ip`, `--compute-mode`. `az databricks workspace update` for tier, public network access, NSG rules, storage firewall. Handy in pipelines and for one-off changes, **not a source of truth**. |

**Bicep example: classic workspace with VNet injection** (from the lesson, trimmed):

```bicep
var mrgSuffix = uniqueString(workspaceName, resourceGroup().id)
var managedResourceGroupId = '${subscription().id}/resourceGroups/databricks-rg-${workspaceName}-${mrgSuffix}'

resource workspace 'Microsoft.Databricks/workspaces@2026-01-01' = {
  name: workspaceName
  location: location
  sku: { name: 'premium' }
  properties: {
    computeMode: 'Hybrid'                 // 'Serverless' for a serverless workspace
    managedResourceGroupId: managedResourceGroupId
    parameters: {
      customVirtualNetworkId: { value: vnetId }
      customPublicSubnetName: { value: publicSubnetName }   // e.g. snet-dbx-host
      customPrivateSubnetName: { value: privateSubnetName } // e.g. snet-dbx-container
      enableNoPublicIp: { value: true }   // secure cluster connectivity (default true)
    }
    publicNetworkAccess: 'Enabled'
    requiredNsgRules: 'AllRules'
  }
}

output workspaceUrl string = workspace.properties.workspaceUrl
output workspaceId string = workspace.properties.workspaceId
```

**There's no single winning tool.** The 1.7 demo deploys the same workspace with Bicep, then shows the Terraform/OpenTofu, stackql-deploy, and Pulumi equivalents. **Workspace-level content (jobs, pipelines, schemas, volumes) uses Declarative Automation Bundles** whichever tool you pick.

### D. Creation-time decisions ⚠️

| Fixed for life | One-way or preview only | Changeable later |
|---|---|---|
| Region, workspace name, `computeMode` | **Managed VNet → VNet injection**: a **Public Preview** ARM update (API `2026-01-01`) with **every cluster stopped**. Terraform doesn't support it, an existing back-end Private Link connection breaks and must be rebuilt, and there's no documented way back. | Pricing tier (upgrade), public network access, required NSG rules |
| Managed RG name and workspace storage account name | **Secure cluster connectivity** can be turned on later (VNet injection required; the update can take more than 15 minutes). Turning it off is supported only as a temporary rollback. | Managed services and managed disk customer-managed keys |
| **Double (infrastructure) encryption** | **Compliance security profile**: treat it as one-way (Section 3 covers the add-on) | Workspace storage firewall and its access connector, diagnostic settings, tags |
| Managed VNet address prefix; injected subnet CIDR ranges | | NCC attachment, network policy, metastore assignment (account console) |

**Announcements that change the defaults:** Databricks has announced that **new classic workspaces will require VNet injection** (managed VNet creation is being deprecated) and that **SCC will become mandatory**, with timelines to follow. **Design every new classic workspace with VNet injection and SCC on**, and choose subnet sizes (Section 3) before you deploy.

> Exam angles:
> - "Deployment failed, workspace shows Failed" → delete and redeploy. Don't try to repair it.
> - "Need a serverless workspace from Terraform" → `azurerm` can't set `computeMode` yet. Use **Bicep or `azapi`**.
> - "Want double encryption on an existing workspace" → not possible. It's creation-only, so you need a new workspace.
> - "User's Contributor role removed, but they're still workspace admin" → expected. The creator stays the first workspace admin.
> - "Deploy fails in a brand new subscription" → check resource provider registration and vCPU quota. Or it's a Free Trial subscription.

### Key takeaways

- Contributor or Owner (or a custom role), plus registered resource providers and vCPU quota. Free Trial subscriptions can't deploy.
- Learn the `Microsoft.Databricks/workspaces` properties once: `computeMode`, `managedResourceGroupId`, `parameters`, `publicNetworkAccess`, `requiredNsgRules`, `encryption`.
- The portal is for learning, Bicep is the native declaration, and Terraform/OpenTofu, stackql-deploy, and Pulumi are equivalent alternatives.
- Fixed for life: region, `computeMode`, managed RG name, storage account name, double encryption, subnet CIDRs.
- Design new classic workspaces with VNet injection and SCC on. Both are becoming mandatory.

### Course vs docs

- No conflicts with DP-750 or with 1.1/1.2. The DP-750 CLI example (`az databricks workspace create ... --sku premium`) still works. This lesson adds `--compute-mode` and the networking flags.
- Things that will go stale, so recheck before relying on them: the Terraform `azurerm` gap for `computeMode`, the "Public Preview" status of the managed VNet → VNet injection migration, the `databricks` provider version (1.140), and the announced timelines for mandatory VNet injection and SCC.

### Official references (from the lesson, checked 2026-10-27)

- [Workspace administration](https://learn.microsoft.com/en-us/azure/databricks/admin/workspace/)
- [Create a workspace](https://learn.microsoft.com/en-us/azure/databricks/admin/workspace/create-workspace)
- [Deploy with Bicep](https://learn.microsoft.com/en-us/azure/databricks/admin/workspace/bicep)
- [Deploy with Terraform](https://learn.microsoft.com/en-us/azure/databricks/dev-tools/terraform/azure-workspace)
- [Update workspace network configuration](https://learn.microsoft.com/en-us/azure/databricks/security/network/classic/update-workspaces)

### ⚡ Quick review

- [ ] Which Azure roles can create a workspace? What extra role does VNet injection need?
- [ ] Which 5 resource providers must be registered?
- [ ] What happens to the creator's admin rights if their Azure role is removed?
- [ ] What should you do with a workspace in the Failed state?
- [ ] Which API version first has `computeMode`? What does `enableNoPublicIp` control?
- [ ] Which property values give back-end Private Link?
- [ ] Why can't the `azurerm` provider create a serverless workspace today? What do you use instead?
- [ ] List 4 things that are fixed for life, and 3 that can be changed later.
- [ ] What are the conditions and catches of migrating a managed VNet to VNet injection?

---

## 1.4 Account Administration on Azure

Lecture, 12 min. Theme: *the tenant-bound account, and a bill that lives in Azure.*

Objectives: bootstrap and operate the Azure Databricks account; find billing in Azure, not in the account console.

### A. Account bootstrap by a Global Administrator ⚠️

**Bootstrap flow:**
1. A **Microsoft Entra ID Global Administrator** signs in to **`accounts.azuredatabricks.net`** with Entra.
2. The **first account admin role is created automatically** for them.
3. They **delegate**: User management → user → Roles tab → **Account admin**.
4. **Remove the Global Admin from the Databricks account.**

**Details:**
- **Tenant binding:** the Databricks account is tied to the **Microsoft Entra tenant ID**. **Workspaces can't move to another tenant.** If you move the Azure resource, it stays in the original account. If you belong to several tenants, the console opens in your **default tenant**.
- **First admin:** **only an Entra Global Administrator** can establish the first account admin, just by signing in to the account console. **Additional account admins need no Entra role at all.**
- **Workspace admins:** anyone with **Azure Contributor or Owner** (at subscription level, or the documented custom role) becomes a **workspace admin on first login**, and keeps it even if the Azure role is removed later. The creator is added automatically.
- **Identity defaults:** **identity federation is on for every new workspace and can't be turned off.** **Automatic identity management** is on for accounts created after **August 1, 2025** (Section 2).

> Exam angles:
> - "Who can set up the first account admin?" → only an Entra **Global Administrator**. After that, account admins need no Entra role.
> - "Least privilege after bootstrap" → delegate account admin, then remove the Global Admin from the Databricks account.
> - "Move a workspace to a different Entra tenant" → not possible.

### B. The account console

Account console = `accounts.azuredatabricks.net` (one per Entra tenant; the account ID is under the username menu).

| Area | What's there |
|---|---|
| **Workspaces** | Every workspace in the tenant with its tier and region. **Update workspace** attaches an **NCC**. The **Network policy** section assigns an egress policy. The **Add-ons** tab enables Mission Critical. |
| **User management** | Users, groups, and service principals at account level. The **Roles** tab grants account admin. Choose **Databricks-managed vs Entra-managed** service principals. |
| **Catalog** | **Metastores per region**: create, assign to workspaces, set the metastore admin, manage configuration |
| **Usage** | Importable usage dashboards (by product, SKU, and custom tag), reading from **`system.billing.usage`** and **`list_prices`**. **Budgets** tab with up to **four email thresholds per budget**. The older DBU usage report **leaves out serverless usage**. **No invoices**: those are in Azure. |
| **Security** | Identity provider setup (the **automatic identity management** toggle, account-wide **federation policies**), Token report, and **Networking**: NCCs plus context-based ingress and egress control policies |
| **Settings, Previews, Resilience** | Feature enablement (the **serverless toggle**, partner-powered AI features with an **Enforce** option, and the separate **ADI Services** integration setting for eligible accounts, see 2.5), account nickname, **preview features**, and **Managed DR** failover groups and stable URLs |

**Billing lives in Azure** ⚠️
- **There's no subscription or billing section in the Azure account console.**
- Usage and Budgets show **DBUs at list price** and **ignore credits and negotiated discounts**.
- Invoiced cost, reservations, and pre-purchase (DBCU) drawdown are in **Azure Cost Management + Billing**. Lesson 1.6 reconciles them against system tables.

> Exam trap: "Finance wants the actual invoiced Databricks cost with discounts" → **Azure Cost Management + Billing**, not the account console's Usage page (list price only).

### C. Platform limits ⚠️

| Object | Limit | Scope |
|---|---|---|
| **Network connectivity configurations (NCCs)** | **20 per region** | Account |
| **Private endpoints** (NCC private endpoint rules) | **100 per region, across all NCCs** | Account |
| **Workspaces per NCC** | **50** | Account |
| IP access list entries | **1,000** IP or CIDR values for the account console; 1,000 per workspace | Account and workspace |
| Users and service principals | 10,000 per account and per workspace | Account and workspace |
| Groups | 5,000 per account; 1,500 direct memberships; 10 nesting levels | Account |
| **Metastores** | **One per region** | Account |
| Managed DR | 100 failover groups per account; 1 per workspace; 10 catalogs each | Account |
| Budgets | 1,000 per account | Account |
| Secret scopes, SQL warehouses, saved jobs | 1,000 scopes, 1,000 warehouses, 12,000 jobs per workspace | Workspace |

**No published workspace-count limit.** Databricks doesn't publish a fixed number of workspaces per account on Azure. In practice, what limits sprawl is **Azure quotas** and the **per-region NCC and private endpoint caps**.

### D. Where Azure quotas intersect

| Compute | Storage and networking | Design consequence |
|---|---|---|
| **vCPU quota per VM family per region** limits cluster size and count. Request increases before go-live. | **250 storage accounts per region per subscription** by default (1,000 on request) | **One subscription per environment** keeps Azure quotas, RBAC, and Azure Policy aligned with the workspace boundary |
| Around **25,000 VMs per subscription per region** | **980 resource groups per subscription.** Each classic workspace creates a managed RG, while its parent RG can be reused. | **Private endpoint and NCC caps are per Databricks account region, not per subscription.** Adding subscriptions doesn't raise them. |
| The 800-resources-per-type resource group limit has exceptions: VMs, managed disks, and NICs are exempt. Their service and subscription quotas still apply. | Public IPs, NAT gateways, VNets, and private endpoints have their own per-subscription networking limits. Check the Azure subscription limits page when planning hub-and-spoke. | **Serverless workspaces avoid most compute and networking quotas entirely** |

> Exam angles:
> - "Need a 21st NCC in East US" or "more than 100 private endpoints in a region" → the account-level cap. More subscriptions won't help.
> - "Second metastore in the same region" → not allowed. One metastore per region; share it across workspaces.
> - "How many workspaces can an account have?" → no published limit. Azure quotas and NCC caps limit it in practice.

### Key takeaways

- One Databricks account per Entra tenant. An Entra Global Administrator creates the first account admin by signing in to `accounts.azuredatabricks.net`.
- Azure Contributor or Owner on the workspace resource makes you workspace admin on first login.
- The account console governs workspaces, identities, metastores, usage and budgets, security, and resilience. Invoices are in Azure Cost Management.
- Limits to remember: 20 NCCs and 100 private endpoints per region, 50 workspaces per NCC, 1,000 IP access list values, one metastore per region.
- Azure quotas (vCPU, storage accounts, resource groups, networking) shape estate design. One subscription per environment keeps them aligned.

### Course vs docs

- No conflicts with DP-750 or earlier lessons. This lesson matches 1.1 (one account per Entra tenant, no separate Databricks bill) and 1.3 (the creator stays workspace admin).
- The "ADI Services" phrase from 1.1 turns out to be a real setting (see the note under 1.1).
- Limits change often. Recheck [Resource limits](https://learn.microsoft.com/en-us/azure/databricks/resources/limits) before quoting a number to a customer.

### Official references (from the lesson, checked 2026-10-27)

- [Admin concepts](https://learn.microsoft.com/en-us/azure/databricks/admin/admin-concepts)
- [Account settings](https://learn.microsoft.com/en-us/azure/databricks/admin/account-settings/)
- [Resource limits](https://learn.microsoft.com/en-us/azure/databricks/resources/limits)
- [Managed disaster recovery](https://learn.microsoft.com/en-us/azure/databricks/admin/managed-disaster-recovery)
- [Budgets](https://learn.microsoft.com/en-us/azure/databricks/admin/account-settings/budgets)
- [Azure subscription and service limits](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/azure-subscription-service-limits)
- [Resources not limited to 800 per resource group](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/resources-without-resource-group-limit)
- [ADI](https://docs.databricks.com/adi)
- [Genie Code CLI](https://learn.microsoft.com/en-us/azure/databricks/genie-code/genie-code-cli)
- [Serverless Private Link](https://learn.microsoft.com/en-us/azure/databricks/security/network/serverless-network-security/serverless-private-link)
- [Create a workspace](https://learn.microsoft.com/en-us/azure/databricks/admin/workspace/create-workspace)

### ⚡ Quick review

- [ ] Who can create the first account admin? Do later account admins need an Entra role?
- [ ] What are the 4 bootstrap steps, and what's the last one?
- [ ] Can a workspace move to another Entra tenant?
- [ ] Which Azure role makes someone a workspace admin automatically?
- [ ] Where do you see invoiced cost with discounts and DBCU drawdown? What does the Usage page show instead?
- [ ] Which usage report leaves out serverless?
- [ ] NCCs per region, private endpoints per region, workspaces per NCC, metastores per region?
- [ ] Why doesn't adding Azure subscriptions raise the NCC or private endpoint caps?
- [ ] Why does one subscription per environment help?

---

## 1.5 Unity Catalog Setup on Azure

Lecture, 12 min. Theme: *metastore, access connector, and the abfss path.*

Objectives: create metastores and access connectors; wire storage credentials and external locations on ADLS Gen2.

UC basics (three-level namespace, managed vs external tables, GRANTs, Purview) are in [DP-750 LP1-M1 §6](../../dp-750/notes/LP1-M1-explore-azure-databricks.md#6-governance-unity-catalog-and-microsoft-purview). This lesson is about the Azure plumbing: how UC reaches your storage.

### A. The regional metastore ⚠️

- **Automatic UC enablement** for new workspaces began rolling out on **November 9, 2023**. Verify the metastore assignment of the actual workspace rather than assuming.
- Automatic enablement creates the regional metastore **without metastore-level managed storage** ("no root storage by design"). Each workspace gets a **workspace catalog** whose managed location is the **workspace storage account** (`dbstorage`).
- **One metastore per region.** Attach every workspace in that region to it.

**How the pieces chain together** (lesson example):

```
Metastore (australiaeast) ── no root storage
├── Workspace catalog ── managed location in the dbstorage account
└── Catalog: lakeside_claims ── managed location abfss://claims@lakesidelake.dfs.core.windows.net/managed
        └── External location: claims_landing ── abfss://landing@lakesidelake.dfs.core.windows.net/
              └── Storage credential ── Azure managed identity via the access connector
                    └── Access Connector for Azure Databricks (Microsoft.Databricks/accessConnectors)
                          └── [Storage Blob Data Contributor] → lakesidelake (ADLS Gen2, hierarchical namespace)
```

**Details:**
- **The metastore admin is optional** on auto-enabled metastores. **Workspace admins** receive CREATE CATALOG, CREATE EXTERNAL LOCATION, CREATE STORAGE CREDENTIAL, CREATE SERVICE CREDENTIAL, CREATE CONNECTION, and related privileges **through a system group**.
- **Manual creation** (account console → Catalog → Create metastore) asks for a name, region, an **optional** ADLS Gen2 path (the `abfss://` prefix is added for you), and an **optional** access connector ID.
- ⚠️ **Assignment is one-way: enabling a workspace for UC can't be reversed.** Attach it through the metastore's **Workspaces** tab or with `databricks account metastore-assignments create`.
- **Design guidance:** assign **managed storage at catalog level** for isolation. Leave the metastore **without root storage**, so a forgotten location fails fast instead of landing data in a shared account.

### B. The access connector ⚠️

The **Access Connector for Azure Databricks** is a first-party Azure resource that **wraps a managed identity**, so UC can reach your storage **without secrets**. It can carry a **system-assigned** identity, one or more **user-assigned** identities, or both. **Put it in the same region as the storage account.**

| Step | Who | What |
|---|---|---|
| 1. Create the connector | **Contributor or Owner on the resource group** | `Microsoft.Databricks/accessConnectors` with `identity.type = SystemAssigned` (or attach a user-assigned identity) |
| 2. Grant storage access | **Owner or User Access Administrator on the storage account** | **Storage Blob Data Contributor** on the account, **or** Storage Blob Delegator on the account plus Storage Blob Data Contributor on a container |
| 3. Grant file-event access | Same | **Storage Queue Data Contributor**, **Storage Account Contributor** (so Databricks can create the queue), and **EventGrid Data Contributor** on the resource group |
| 4. Create the storage credential | **CREATE STORAGE CREDENTIAL** on the metastore | Credential type **Azure Managed Identity**, with the connector's **resource ID** (and, for user-assigned, the identity's resource ID) |
| 5. Create external locations | **CREATE EXTERNAL LOCATION** on the metastore **and** the credential | `CREATE EXTERNAL LOCATION name URL 'abfss://container@account.dfs.core.windows.net/path' WITH (STORAGE CREDENTIAL cred)` |

**Why managed identities and not service principals:**
- No secrets to store or rotate.
- **Only a managed identity can be admitted through a storage firewall** as a trusted service or resource instance.
- **Service principal storage credentials are documented as legacy.** Section 2 covers user-assigned identity patterns and firewall traversal in depth.

> Exam angles:
> - "Who assigns Storage Blob Data Contributor to the connector?" → someone with **Owner or User Access Administrator** on the storage account. Contributor isn't enough to assign roles.
> - "Storage account has a firewall; UC access fails with a service principal credential" → switch to an **access connector (managed identity)**. Only that can pass the firewall as a trusted resource instance.
> - "Auto Loader file-notification mode can't create its queue" → missing **Storage Queue Data Contributor / Storage Account Contributor / EventGrid Data Contributor**.

### C. External locations, overlap, and CORS

**Path rules**
- **ADLS Gen2 with the hierarchical namespace.** Azure Blob Storage (no HNS) isn't supported. **OneLake external locations are in Beta** (ID-based abfss paths, Databricks Runtime 18.1 or above, or serverless).
- Paths use standard ASCII. **Immutable (WORM) containers are rejected.**
- **Catalog and schema managed locations must sit inside an external location.** Managed locations can't overlap external tables or volumes, and **external locations can't overlap each other**.

**CORS for volume uploads**
- **Uploading files to managed volumes from the UI needs a CORS rule** on the storage account's **Blob** service.
- Allowed origins `https://*.azuredatabricks.net`, allowed methods `PUT`, allowed headers `x-ms-blob-type`, exposed headers blank, max age `1800`.
- Set it in the portal under **Settings → Resource sharing (CORS)**.

**File events**
- **On by default for new external locations.** They power Auto Loader and file-arrival triggers without per-pipeline wiring.
- **Automatic** mode creates the queue and Event Grid subscription for you. **Provided** mode brings your own.
- **Premium storage accounts can't use automatic mode** (queues need Standard).

### D. Default storage or your own ADLS

| Choose | When |
|---|---|
| **Default storage** (serverless workspaces) | Serverless-only estate, teams without Azure infrastructure rights, fastest start. BI over ODBC/JDBC and the Files API still work. |
| **Customer-managed ADLS Gen2** via access connector | **Classic compute must read the data**, external engines need file-level or credential-vended access, or you need your own **CMK, firewall, private endpoints**, or other tools that write to the lake |
| **Both** | Workspace catalog and scratch work on default storage; governed domain catalogs with managed locations on your storage accounts |

(Ties back to 1.2: classic compute can't read default storage.)

### Key takeaways

- One metastore per region. Automatic enablement rollout began November 9, 2023, and creates the metastore without root storage. The workspace catalog lives in the workspace storage account.
- The access connector wraps a system- or user-assigned managed identity. Grant Storage Blob Data Contributor, plus the Queue, Storage Account, and EventGrid roles for file events.
- Storage credential (Azure Managed Identity, connector resource ID), then external locations on `abfss://` paths. Locations can't overlap.
- A CORS rule on the Blob service (origin `https://*.azuredatabricks.net`, PUT, `x-ms-blob-type`, max age 1800) enables UI uploads to managed volumes.
- Default storage for serverless quick starts. Customer ADLS Gen2 when classic compute, external engines, or your own network and key controls are involved.

### Course vs docs

- **Wording difference with DP-750:** DP-750 says metastores are "**typically** one per region". This course (and lesson 1.4's limits table) says **one per region** as a hard limit. For the exam, treat it as a limit: one metastore per region per account.
- Both agree that automatic UC enablement started for workspaces created after November 9, 2023.
- OneLake external locations are Beta. Expect this to change.

### Official references (from the lesson, checked 2026-10-27)

- [Create a metastore](https://learn.microsoft.com/en-us/azure/databricks/data-governance/unity-catalog/create-metastore)
- [Azure managed identities for UC](https://learn.microsoft.com/en-us/azure/databricks/connect/unity-catalog/cloud-storage/azure-managed-identities)
- [External locations on ADLS](https://learn.microsoft.com/en-us/azure/databricks/connect/unity-catalog/cloud-storage/external-locations-adls)
- [Manage external locations](https://learn.microsoft.com/en-us/azure/databricks/connect/unity-catalog/cloud-storage/manage-external-locations)
- [Storage CORS](https://learn.microsoft.com/en-us/azure/databricks/data-governance/unity-catalog/storage-cors)
- [Default storage](https://learn.microsoft.com/en-us/azure/databricks/storage/default-storage)
- [Standard tier](https://learn.microsoft.com/en-us/azure/databricks/admin/account-settings/standard-tier)
- [Enable workspaces for UC](https://learn.microsoft.com/en-us/azure/databricks/data-governance/unity-catalog/enable-workspaces)
- [OneLake external locations](https://learn.microsoft.com/en-us/azure/databricks/connect/unity-catalog/cloud-storage/external-locations-onelake)

### ⚡ Quick review

- [ ] What does automatic UC enablement create, and what does it leave out?
- [ ] Who gets CREATE CATALOG etc. on an auto-enabled metastore if there's no metastore admin?
- [ ] Can you un-assign a workspace from UC?
- [ ] Why leave the metastore without root storage?
- [ ] Put the 5 access connector steps in order, with who does each one.
- [ ] Which 3 extra roles do file events need? Which storage tier can't use automatic mode?
- [ ] Why managed identity over service principal for storage credentials?
- [ ] What are the CORS settings for volume uploads (origin, method, header, max age)?
- [ ] Which storage types are rejected for external locations?
- [ ] When must you use your own ADLS instead of default storage?

---

## 1.6 Operating Azure Databricks Estates

Lecture, 10 min. Theme: *from one workspace to a governed, costed fleet.*

Objectives: reconcile Azure Cost Management with the billing system tables; apply landing zone and DR patterns.

### A. Interpreting billing information ⚠️

A classic cluster run (for example, 1 driver + 4 workers for 2 hours) produces **two kinds of meters**:
- **Infrastructure meters**: Virtual Machines, Managed Disks, Storage, NAT gateway, bandwidth → **Azure Cost Management**
- **DBU meters**: the Azure Databricks service, per SKU (all-purpose, jobs, SQL) → **Azure Cost Management** *and* **`system.billing.usage`**

**The two cost lenses:**

| Lens | What it shows |
|---|---|
| **Azure Cost Management** (the invoice view) | DBU meters **plus** the VM, disk, storage, and network meters of the managed RG, in your billing currency, **after reservations and negotiated rates**. **Serverless usage shows up as DBU meters only.** |
| **System tables** (the platform view) | **`system.billing.usage`** has DBU quantities per SKU with `custom_tags`, `usage_metadata` (cluster, job, warehouse IDs), and `identity_metadata`. **`system.billing.list_prices`** turns quantities into list-price dollars. Records arrive within **about 12 hours**. |
| Account console Usage and Budgets | **List prices**, ignoring credits and discounts. The pre-purchase plan shows on-demand charges as **zero** in Azure usage data when covered. |

**Review both lenses together.** Cheap DBUs on expensive VM SKUs, or an idle NAT gateway, only show up when you join them. **Tags are the join key.**

### B. Tag propagation rules ⚠️

Workspace, pool, and cluster tags are combined by Azure Databricks and **propagated to the Azure VMs in the managed RG**. That's what lets Cost analysis group infrastructure spend the same way system tables group DBUs.

**Default tags**
- All-purpose and jobs compute: `Vendor` = Databricks, `ClusterId`, `ClusterName`, `Creator`, plus `RunName` and `JobId` for jobs
- SQL warehouses: `Vendor`, `ClusterId`, `SqlEndpointId`, `Creator`
- Pools: `Vendor`, `DatabricksInstancePoolCreatorId`, `DatabricksInstancePoolId`

**The `x_` prefix rule**
- A custom tag whose key collides with a default tag is **renamed with an `x_` prefix** on propagation (`vendor` becomes `x_vendor`). The default tag keeps its name.
- Conflicting tags set through **compute policies don't resolve**: cluster creation fails with an invalid settings error.
- Cluster tags that conflict with pool tags **block cluster creation**.

**The 50-tag cap**
- **Azure allows at most 50 tags per resource.**
- Over the cap, **`x_`-prefixed tags are dropped first** in alphabetical order, then the remaining tags alphabetically.
- ⚠️ **Clusters launched from pools carry workspace and pool tags only. Cluster tags aren't propagated.**

**Timing and characters**
- **Workspace tag changes take up to an hour** to propagate. Cluster and pool tag changes apply after a **restart or pool expansion**.
- Allowed characters: letters, spaces, numbers, and `+ - = . _ : / @`.
- ⚠️ **Serverless usage never lands on a VM tag.** Use **serverless usage policies** (Public Preview) and `system.billing.usage` for it.

> Exam angles:
> - "Cost center tag missing on VMs for pool-backed clusters" → pool clusters only get **workspace and pool** tags. Put the tag on the pool or workspace.
> - "Team tagged `vendor=acme`; on the VM it's `x_vendor`" → expected, because it collided with the default `Vendor` tag.
> - "How do you attribute serverless spend by team?" → **serverless usage policies** + `system.billing.usage`. There are no VMs to tag.
> - "Finance's numbers don't match the account console" → console is list price; Cost Management is after discounts and reservations.

### C. Estate topology and landing zones

**A typical Azure estate** (lesson example):

```
Databricks account (one per tenant)
├── Metastore australiaeast
└── NCC australiaeast
     │
     ├── Subscription: lakeside-dev   (metastore assignment)
     │     ├── Workspace lakeside-dev      classic, managed VNet
     │     └── Workspace lakeside-sandbox  serverless
     ├── Subscription: lakeside-prod  (metastore + NCC assignment)
     │     └── Workspace lakeside-prod     VNet injection, Private Link, CMK ──┐
     └── Subscription: connectivity hub                                       │
           └── Azure Firewall, ExpressRoute, private DNS zones  ◄─────────────┘
```

| Cloud Adoption Framework | Well-Architected service guide | Workspace strategy |
|---|---|---|
| The CAF cloud-scale analytics scenario (which included Azure Databricks pages) **now redirects to the Fabric-centred *Data for AI and Analytics* guidance**, which treats Azure Databricks as an integrated Azure service with its own pricing. **The landing-zone pattern still stands**: workspaces in a **workload landing zone**, the connectivity hub and identity in the **platform landing zone**. | *Architecture best practices for Azure Databricks* covers **reliability** (multi-region and DR), **security** (UC, CMK, SCC, Private Link, audit logs, one workspace per environment), **cost**, **operational excellence** (serverless workspaces, ARM and Bicep), and **performance**. ⚠️ Read its tier guidance against the Standard tier retirement (1.2). | **Split by SDLC environment with a subscription each**, share the regional metastore, **bind production catalogs to production workspaces**, and **isolate teams at catalog level rather than by workspace sprawl**. |

### D. Disaster recovery ⚠️

| DIY active-passive | Managed Disaster Recovery | Managed DR limits to know |
|---|---|---|
| A second workspace in another region, normally in the same account and tenant. **Databricks now points you to Managed DR first**, and keeps the DIY path for what Managed DR doesn't replicate, for active-active, and for cross-cloud. | **Gated**: apply through your account team. Needs **Premium in both regions** and the **Mission Critical add-on on both workspaces**. | **Not replicated:** managed volume data, secrets, models, serving endpoints, vector search indexes, Delta shares, and published dashboards. **Materialized views, streaming tables, and Lakeflow pipelines replicate only in Private Preview.** |
| Replicate tables with **Deep Clone**. ⚠️ **Geo-redundant storage is not a substitute for the workspace storage account.** Configuration with IaC, code with Git. | Needs serverless compute, account-level SSO with AIM or SCIM, a **secondary metastore**, and an **NCC and access connector in the secondary region**. | **Secondary catalogs are read-only.** The initial bootstrap can take **up to five days** for large workspaces. |
| **You own** the runbooks, the RPO and RTO testing, and the drift. | **Failover groups and stable URLs** in the account console under **Resilience**. **21 Azure regions**, with some pairs ruled out (**Australia East can't pair with Australia Southeast**). | **One failover group per workspace, ten catalogs per group.** |

> Exam angles:
> - "Is GRS on the workspace storage account enough for DR?" → **no**. Use Deep Clone or Managed DR.
> - "Managed DR prerequisites" → Premium in both regions, Mission Critical add-on on both workspaces, serverless, account-level SSO with AIM/SCIM, secondary metastore, NCC + access connector in the secondary region, and approval through the account team.
> - "Need secrets and serving endpoints in the DR region" → Managed DR doesn't replicate them. Handle with IaC (DIY).

### Key takeaways

- Azure Cost Management holds the invoice (DBU meters plus VM, disk, storage, network). `system.billing.usage` holds DBUs with tags and metadata. Join them by tag.
- Tags propagate to Azure VMs. Conflicting custom keys get an `x_` prefix, and Azure caps resources at 50 tags.
- Pool-launched clusters carry pool and workspace tags only. Serverless usage is tagged through usage policies.
- One subscription per environment, one metastore per region, catalog-level isolation, hub connectivity in a platform landing zone.
- Databricks recommends Managed DR for cross-region recovery. It's gated, needs Mission Critical on both workspaces, and has scope limits, so DIY active-passive stays for the rest.

### Course vs docs

- **CAF change:** the course notes that the CAF cloud-scale analytics guidance for Azure Databricks now redirects to the Fabric-centred *Data for AI and Analytics* guidance. Older study material that points to the cloud-scale analytics scenario is out of date, but the landing-zone pattern hasn't changed.
- **Well-Architected tier guidance** may still mention Standard tier. The course says to read it against the Standard retirement in 1.2.
- **Things likely to change:** serverless usage policies (Public Preview), Managed DR replication of MVs/streaming tables/pipelines (Private Preview), and the Managed DR region list (21 regions).
- Fits with 1.4: account console Usage and Budgets are list price only, and the real invoice is in Azure Cost Management.

### Official references (from the lesson, checked 2026-10-27)

- [Billing system tables](https://learn.microsoft.com/en-us/azure/databricks/admin/system-tables/billing)
- [Usage detail tags](https://learn.microsoft.com/en-us/azure/databricks/admin/account-settings/usage-detail-tags)
- [Usage (account settings)](https://learn.microsoft.com/en-us/azure/databricks/admin/account-settings/usage)
- [Managed disaster recovery](https://learn.microsoft.com/en-us/azure/databricks/admin/managed-disaster-recovery)
- [Best practices](https://learn.microsoft.com/en-us/azure/databricks/getting-started/best-practices)
- [Well-Architected service guide: Azure Databricks](https://learn.microsoft.com/en-us/azure/well-architected/service-guides/azure-databricks)
- [CAF cloud-scale analytics: data ingestion](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/scenarios/cloud-scale-analytics/best-practices/data-ingestion)
- [CAF landing zones](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/landing-zone/)

### ⚡ Quick review

- [ ] What does Azure Cost Management show that `system.billing.usage` doesn't, and vice versa?
- [ ] How fast do records arrive in `system.billing.usage`? Which table converts to list-price dollars?
- [ ] What's the default tag set for jobs compute, SQL warehouses, and pools?
- [ ] What happens when a custom tag key collides with a default tag? With a compute policy tag? With a pool tag?
- [ ] What's the Azure tag cap per resource, and which tags are dropped first?
- [ ] Which tags does a pool-launched cluster carry?
- [ ] How long do workspace tag changes take? How do you tag serverless spend?
- [ ] What's the recommended workspace and subscription strategy?
- [ ] Managed DR: prerequisites, what isn't replicated, and limits per workspace and group
- [ ] Why isn't GRS a DR plan for the workspace storage account?

---

<!-- Next: 1.7 Demo: Azure Databricks Workspace Administration Fundamentals -->
