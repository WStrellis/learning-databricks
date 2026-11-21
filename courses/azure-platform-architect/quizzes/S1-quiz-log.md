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
