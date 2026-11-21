# LP1 · M1: Explore Azure Databricks

Source: https://learn.microsoft.com/en-us/training/modules/explore-azure-databricks/ (8 units, about 68 min, updated 2026-10-26)
Lab: [01 Explore Azure Databricks](https://microsoftlearning.github.io/DP-750T00-Implement-Data-Engineering-Solutions-using-Azure-Databricks/Instructions/Labs/01-explore-azure-databricks.html) (45 min, needs an Azure subscription with admin rights)

---

## 1. Big picture

- Azure Databricks is a unified platform for data engineering, data science, ML, and BI, built on **Apache Spark**.
- **Lakehouse** combines lake and warehouse strengths: scalable storage plus ML and BI on one copy of the data. That gives a single source of truth, no duplicate costs, and current data.
- **Medallion architecture** refines data in layers that build on each other (bronze → silver → gold).
- The two key lakehouse technologies:
  - **Delta Lake** is the storage layer. It provides ACID and schema enforcement.
  - **Unity Catalog** provides unified, fine-grained governance for data and AI.

## 2. Workspaces: creating one

**Workspace** = an Azure Databricks deployment that gives a set of users a unified environment for Databricks assets. You need one in your subscription before you can do anything.

**Four ways to create one:**
| Method | Command / tool |
|---|---|
| Portal | Azure portal UI |
| IaC | ARM, **Bicep**, or **Terraform** template |
| PowerShell | `New-AzDatabricksWorkspace -Name -ResourceGroupName -Location -ManagedResourceGroupName -Sku premium` |
| Azure CLI | `az databricks workspace create --resource-group --name --location --sku premium` |

**Settings:** workspace name, region, pricing tier, workspace type, and an optional **Managed Resource Group** name. Azure auto-creates the managed resource group to hold the infrastructure it manages for the workspace.

### Pricing tiers ⚠️
| Tier | Notes |
|---|---|
| **Premium** | RBAC, **Unity Catalog, Databricks SQL, Mosaic AI, serverless**, Genie Code, and other enterprise features |
| **Trial** | **14-day** free trial of a Premium-level workspace |
| ~~Standard~~ | **Not available for new workspaces as of April 1, 2026.** All new workspaces are Premium. |

> Exam trap: older questions may say that SQL warehouses, Databricks SQL, or Unity Catalog **need Premium**. That is still true, but now every new workspace is Premium anyway.

### Workspace types
| Type | What it is |
|---|---|
| **Serverless** | Comes with serverless compute and **default storage**. Recommended for the simplest quick start. |
| **Hybrid (Classic)** | Compute and storage are provisioned **in your own Azure subscription** |

**Free Edition** is different from the Azure Trial. It has no time limit and costs nothing, but it has daily usage limits and **no classic compute**.

## 3. Workspace UI

- **Launch Workspace** button in the Azure portal opens the web UI.
- **Homepage:** Ask Genie, plus suggested, favorites, and popular objects.
- **Sidebar:** Workspace, Recents, Catalog, Jobs & Pipelines, Compute, Discover, Marketplace
  - **SQL:** SQL Editor, Queries, Dashboards, Genie Agents, Alerts, Query History, SQL Warehouses
  - **Data Engineering:** Runs, Data Ingestion
  - **AI/ML:** Playground, Agents, AI Gateway, Experiments, Features, Models, Serving
- **+ New** creates objects (notebooks, queries, repos, dashboards, alerts, jobs, pipelines, experiments, models, serving endpoints) and compute (clusters, SQL warehouses, ML endpoints).
- **Search** covers notebooks, queries, dashboards, alerts, files, folders, libraries, **UC tables**, jobs, and repos.
- **UI language:** username → Settings → Preferences.

### Genie Code (formerly **Databricks Assistant**)
- An AI pair programmer that generates, explains, and fixes code in notebooks, dashboards, and files. It can also diagnose job issues, build visualizations, and answer questions from the docs.
- **Agent mode:** plans multi-step work, retrieves assets, runs code, and learns from cell output. It **asks for approval** before running code or editing a notebook.
- Uses **Unity Catalog metadata** (tables, columns, **descriptions**) to personalize answers. That's why good comments matter (domain 2: "descriptions for data discovery").

## 4. Workloads

| Workload | Key points |
|---|---|
| **Data engineering** | Spark in the lakehouse. Python, R, Scala, SQL. Exploration, visualization, pipelines. |
| **Machine learning** | **MLflow** for experiments, reproducibility, and deployment. TensorFlow, PyTorch, scikit-learn. |
| **SQL / BI** | SQL warehouses, SQL editor, dashboards, ad hoc queries. **Needs Premium.** |
| Also | Data science, LLMs, streaming |

## 5. Key concepts

### Workspace
- Organizes notebooks, clusters, jobs, libraries, dashboards, and experiments. Has a web UI and **REST APIs**.
- Supports folders and permissions at different levels.
- Attached to a **Unity Catalog metastore** for governance.
- **Classic workspaces** link to a **managed resource group** that holds the compute, networking, and storage behind the scenes.

### Notebooks
- Combine code, visualizations, and narrative text. Python, R, Scala, and SQL, and you can switch language per cell with **magic commands** (`%sql`, `%python`, `%scala`, `%r`, `%md`).
- Cell types: **code** and **Markdown**. You can run one cell, a group of cells, or all of them.
- Support real-time co-editing and comments. Notebooks can be version-controlled, **scheduled as jobs**, or exported.

### Architecture: control plane vs compute plane ⚠️
| Plane | Where it lives | What it does |
|---|---|---|
| **Control plane** | Databricks account (Databricks-managed) | Backend services and the web app |
| **Compute plane (classic)** | **Your Azure subscription** | Processes data |
| **Compute plane (serverless)** | **Your Azure Databricks account** (Databricks-managed) | Processes data |

- **Cluster** = a **driver** node that coordinates plus one or more **worker** nodes that do the distributed work.
- A cluster can be fixed size or **autoscaling**, which adds and removes workers on demand.

### Compute options
| Option | Use when |
|---|---|
| **Serverless** | You want fast startup, no management, and elastic scaling |
| **Classic** | You need full control of VM size, libraries, and runtime, or consistent performance for specialized workloads |
| **SQL warehouse** | SQL analytics and BI. Available as serverless or classic (plus Pro, see below). |

### SQL warehouse types ⚠️ (likely exam material)
| Type | Photon | Predictive IO | Intelligent Workload Mgmt | Where compute runs | Startup | Use when |
|---|---|---|---|---|---|---|
| **Serverless** | ✅ | ✅ | ✅ | Databricks account | Seconds | Default choice. No idle cost. |
| **Pro** | ✅ | ✅ | ❌ | Your subscription | Minutes | Serverless isn't available, or you need **custom networking** (federation, hybrid/on-prem) |
| **Classic** | ✅ | ❌ | ❌ | Your subscription | **About 4 min** | Basic interactive exploration only |

- SQL warehouses were formerly called **SQL endpoints**. They're compute decoupled from storage. **All Databricks SQL queries run on them.**
- **Databricks SQL** supports ANSI SQL and is **Premium-only**.

### Databricks Runtime (DBR)
- Customized Spark builds with performance improvements and extra libraries. Each version lists its Spark version, release date, and end-of-support date.

| Stage | Meaning |
|---|---|
| Beta | Evaluation only, not for production |
| GA | New features and fixes under the same version for **about 6 months** |
| **LTS** | Stability and security fixes backported for **3 years** |
| End of Support (EoS) | No fixes. **Can't select it** when creating or editing compute. |
| End of Life (EoL) | Retired and unusable |

- **To pick up a runtime maintenance update, restart the cluster.**

### Lakeflow Jobs (formerly Workflows)
- Orchestration for ETL, ML training, and dashboard refreshes.
- **Job** = a container for one or more **tasks** (notebook, Spark job, external code, and so on).
- **Triggers:** schedule (for example, nightly at midnight), **event**, or **manual**.

### Delta Lake
- Open-source storage framework that adds transactions on top of object storage such as **ADLS**. Solves inconsistent data, partial writes, and concurrent access problems.
- **ACID**, **scalable metadata** (lots of files), **versioning, time travel, and rollback**, and **unified batch and streaming** on the same table.
- Query with SQL or the DataFrame API. **Delta is the default table format in Azure Databricks.**

### MLflow
- Open source and covers the end-to-end ML lifecycle: experiments, model management, and dev → prod. Also handles GenAI and agent evaluation.

## 6. Governance: Unity Catalog and Microsoft Purview

**Problem:** data is siloed across sources, formats (Parquet, CSV, Delta), and asset types (files, models, notebooks, dashboards). That raises compliance and security risk, creates lock-in and duplicate costs, and slows decisions.

### Unity Catalog
- Centralizes **access control, discovery, lineage, audit logs, and quality monitoring** for data and AI assets.
- Applies to **all workspaces attached to the same metastore**.
- **Metastore** is the top-level container for metadata and permissions. **Typically one per region**, and many workspaces can share it.
- **Three-level namespace: `catalog.schema.object`**
  - **Catalog** groups assets, usually by **team or environment** (for example `dev`, `prod`)
  - **Schema** (also called database) groups assets by **project or use case**
  - **Objects:** tables (managed or external), views, **volumes**, **functions**, **models**

| | Managed table | External table |
|---|---|---|
| UC manages governance | ✅ | ✅ |
| UC manages storage and lifecycle | ✅ | ❌ (managed externally) |
| Formats | **Delta or Iceberg** | Delta, CSV, JSON, Parquet, … |

- **Access control:** ANSI SQL `GRANT` at the metastore, catalog, schema, table, row, and column levels.
  ```sql
  GRANT CREATE TABLE ON SCHEMA mycatalog.myschema TO `finance-team`;
  ```
- **Discovery:** Catalog Explorer and search, with tags, comments, and **AI-generated descriptions**. Each asset also has lineage, table insights, and **ER diagrams**.
- **Audit and lineage go down to the column level.**
- **UC is enabled automatically for workspaces created after November 9, 2023.**

### Microsoft Purview
- Enterprise governance across on-prem, multicloud, and SaaS: discovery, **classification**, lineage, and access governance.
- With Databricks, Purview **scans** UC metadata into the **Purview Data Map**, its central catalog.
  - Uses the managed **integration runtime** and works over public or **private** networks.
  - Can scan the **whole metastore or selected catalogs**.
  - Extracts metastore, catalogs, schemas, tables and views, and columns.
  - **Auto-classifies** sensitive data with built-in or **custom** rules.
  - Captures lineage, including transformations inside Databricks.
  - Scans run **on demand or on a daily, weekly, or monthly schedule**.
- **Can also scan the legacy workspace-level Hive metastore.**

> Rule of thumb: **Unity Catalog** governs inside Databricks (access, lineage, audit). **Purview** is the org-wide catalog and classification layer that pulls UC metadata in.

---

## ⚡ Quick-review checklist
- [ ] Premium is required for UC, DBSQL, and serverless. Trial lasts 14 days. Standard ended for new workspaces on April 1, 2026.
- [ ] Serverless vs hybrid/classic workspace type
- [ ] Control plane is Databricks-managed. Classic compute runs in your subscription, serverless compute in the Databricks account.
- [ ] Serverless vs Pro vs Classic SQL warehouse (Photon / Predictive IO / IWM)
- [ ] DBR lifecycle: GA about 6 months, LTS 3 years. Restart the cluster to apply maintenance updates.
- [ ] Metastore: one per region, shared by workspaces. `catalog.schema.object`
- [ ] Managed tables are Delta or Iceberg with UC-managed storage. External tables use external storage.
- [ ] UC is on by default after November 9, 2023
- [ ] Purview scans UC (whole metastore or selected catalogs) and the Hive metastore into the Data Map
- [ ] 4 ways to create a workspace: portal, ARM/Bicep/Terraform, PowerShell, CLI

## My notes from the lab
<!-- add your own observations here after doing the lab -->
