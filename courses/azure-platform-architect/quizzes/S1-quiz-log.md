# S1 quiz log

> ⚠️ **Scores for 1.1–1.5 are not reliable.** The "reply in a format like…" example in each quiz matched the real answers, at least in part. Use a fresh Section 1 re-quiz as the real baseline.

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

## 1.4 Account Administration on Azure (2026-11-21)

Score: **6/6**

| # | Topic | Answer | Correct | Result |
|---|---|---|---|---|
| 1 | Order: Global Admin signs in → first account admin auto-created → delegate → remove Global Admin | BDAC | BDAC | ✓ |
| 2 | Invoiced cost with discounts and DBCU drawdown → Azure Cost Management + Billing | C | C | ✓ |
| 3 | 100 private endpoints per region across NCCs is an account cap; subscriptions don't help | B | B | ✓ |
| 4 | Later account admins need no Entra role; Contributor → workspace admin on first login (TWO) | AC | AC | ✓ |
| 5 | One metastore per region → isolate with a catalog | D | D | ✓ |
| 6 | Older DBU usage report leaves out serverless → use `system.billing.usage` dashboards | A | A | ✓ |

### Weak spots

- None this round.

## 1.5 Unity Catalog Setup on Azure (2026-11-23)

Score: **5/6**

| # | Topic | Answer | Correct | Result |
|---|---|---|---|---|
| 1 | Order: access connector → Storage Blob Data Contributor → storage credential → external location | BDAC | BDAC | ✓ |
| 2 | Assigning the storage role needs Owner or User Access Administrator | B | B | ✓ |
| 3 | Storage firewall → access connector (managed identity) as trusted resource instance | C | C | ✓ |
| 4 | Auto-enabled metastore: no root storage; workspace admins get privileges via system group (TWO) | CD | AC | ✗ |
| 5 | UI volume upload fails → CORS rule on the Blob service | D | D | ✓ |
| 6 | Premium storage can't use automatic file events mode | A | A | ✓ |

### Weak spots

- **Auto-enabled metastore defaults (1.5 Q4).** Picked "a Global Administrator must be named metastore admin" (D) instead of "created without root storage" (A). On auto-enabled metastores the **metastore admin is optional**: workspace admins get the key privileges through a system group, and the Global Admin is only needed to bootstrap the *account* (1.4), not the metastore. The metastore has **no root storage by design**; the workspace catalog uses the workspace storage account on classic workspaces, or default storage on serverless workspaces. Re-test in a later quiz.

## 1.6 Operating Azure Databricks Estates (2026-11-23)

Score: **6/6** (first quiz with placeholder-only format instructions, so this score is reliable)

| # | Topic | Answer | Correct | Result |
|---|---|---|---|---|
| 1 | Pool-launched clusters don't carry cluster tags → tag the pool or workspace | C | C | ✓ |
| 2 | Collision with default `Vendor` tag → `x_vendor` | B | B | ✓ |
| 3 | Cost Management is after discounts; serverless = DBU meters only (TWO) | AC | AC | ✓ |
| 4 | NOT a Managed DR requirement: GRS on workspace storage | D | D | ✓ |
| 5 | Serverless spend by team → serverless usage policies + `system.billing.usage` | D | D | ✓ |
| 6 | Over 50 tags → `x_` tags dropped first, alphabetically | A | A | ✓ |

### Weak spots

- None this round.

## Section 1 mixed quiz, 1.1–1.6 (2026-11-24)

Score: **8/10**. First reliable baseline for Section 1 (new questions, placeholder-only format).

| # | Lesson | Topic | Answer | Correct | Result |
|---|---|---|---|---|---|
| 1 | 1.1 | What an Owner can still do with the managed RG (TWO) | AB | AC | ✗ |
| 2 | 1.1 | Account ID location (weak-spot re-test) | C | C | ✓ |
| 3 | 1.2 | Access control after Standard → Premium upgrade | A | A | ✓ |
| 4 | 1.3 | Order: register providers → VNet/subnets → deploy workspace → assign metastore | BCDA | CDAB | ✗ |
| 5 | 1.3 | Fixed for life: managed RG name, region (TWO) | AD | AD | ✓ |
| 6 | 1.4 | Later account admins need no Entra role | C | C | ✓ |
| 7 | 1.4 | 50 workspaces per NCC | D | D | ✓ |
| 8 | 1.5 | No metastore admin → workspace admins via system group (weak-spot re-test) | C | C | ✓ |
| 9 | 1.5 | Catalog managed location inside an external location on ADLS Gen2 (HNS) | A | A | ✓ |
| 10 | 1.6 | Managed DR secondary catalogs are read-only | D | D | ✓ |

### Weak spots

- **Account ID vs workspace ID: confirmed fixed** (2nd correct in a row).
- **Auto-enabled metastore privileges: fixed** (1.5 Q4 miss, now correct).
- **Managed RG deny assignment (Q1).** Picked "add an NSG rule" (B). The deny assignment blocks writes to the NSG and other resources even for Owner. What you *can* do: read and list the deny assignment, delete the whole workspace, and choose the name at creation. Re-test.
- **Deployment order (Q4).** Put "assign the metastore" first. A workspace must exist before it can be assigned to a metastore; prerequisites (providers, then networking) come before the deploy. Re-test with another ordering question.
