# S1 quiz log

## 1.1 Azure Databricks Service Architecture (2026-11-21)

Score: **5/6**

| # | Topic | Answer | Correct | Result |
|---|---|---|---|---|
| 1 | Owner blocked on managed RG NSG → system deny assignment | C | C | ✓ |
| 2 | Where control plane and serverless compute run (TWO) | BD | BD | ✓ |
| 3 | Terraform `managed_resource_group_name` change → recreates workspace | B | B | ✓ |
| 4 | Number in `adb-<n>.<m>.azuredatabricks.net` | B | A | ✗ |
| 5 | 14-day trial: free DBUs only, no serverless workspace or CMK | D | D | ✓ |
| 6 | DBCU pre-purchase via Azure Reservations, purchases final | C | C | ✓ |

### Weak spots

- **Workspace ID vs account ID.** The number after `adb-` in the workspace URL is the **workspace ID** (also after `?o=` on legacy URLs, in the account console's workspace list, and in Terraform outputs). The **account ID** is per Entra tenant, sits under the username menu in the account console, and never appears in a workspace URL. Re-test this next quiz.

## 1.2 Workspace Types and Tiers (2026-11-21)

Score: **5/6**

| # | Topic | Answer | Correct | Result |
|---|---|---|---|---|
| 1 | Scala + ExpressRoute → Hybrid with VNet injection | B | B | ✓ |
| 2 | `computeMode` fixed → new serverless workspace | C | C | ✓ |
| 3 | Serverless supports managed services CMK + front-end PL (TWO) | AD | AD | ✓ |
| 4 | Classic compute can't read default storage | B | B | ✓ |
| 5 | Upgrade VNet-injected Standard → Premium (portal/ARM, access control one-way) | D | D | ✓ |
| 6 | Re-test: where to find the Databricks account ID | A | B | ✗ |

### Weak spots

- **Workspace ID vs account ID: missed twice (1.1 Q4, 1.2 Q6).** Both times the answer picked the number from the workspace URL. The URL number is always the **workspace ID**. The **account ID** is a GUID, never appears in any workspace URL, and is found only in the **account console** (`accounts.azuredatabricks.net`) under the username menu. Keep re-testing until it sticks.

## 1.3 Deploying Workspaces (2026-11-21)

Score: **6/6**

| # | Topic | Answer | Correct | Result |
|---|---|---|---|---|
| 1 | Failed workspace → delete and redeploy | C | C | ✓ |
| 2 | Serverless from Terraform → `azapi` or Bicep (azurerm lacks `computeMode`) | B | B | ✓ |
| 3 | Fixed for life: double encryption, injected subnet CIDRs (TWO) | AD | AD | ✓ |
| 4 | New subscription → register resource providers | A | A | ✓ |
| 5 | Managed VNet → VNet injection: preview, clusters stopped, back-end PL rebuilt, no way back | D | D | ✓ |
| 6 | Re-test #3: account-level Terraform `account_id` = GUID from account console | C | C | ✓ |

### Weak spots

- **Workspace ID vs account ID: fixed on the 3rd try (1.3 Q6).** Re-test once more in a later section to confirm it has stuck.
