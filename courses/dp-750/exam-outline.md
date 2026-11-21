# DP-750 Skills Measured (as of March 11, 2026)

Passing score is 700. Most questions cover GA features, but commonly used Preview features may also appear.
Practice assessment: https://aka.ms/DP750-PA

| Domain | Weight | Learning Path |
|---|---|---|
| 1. Set up and configure an Azure Databricks environment | 15–20% | LP1 (5 modules) |
| 2. Secure and govern Unity Catalog objects | 15–20% | LP2 (2 modules) |
| 3. Prepare and process data | **30–35%** | LP3 (4 modules) |
| 4. Deploy and maintain data pipelines and workloads | **30–35%** | LP4 (4 modules) |

Check off skills as you cover them in notes and labs.

## 1. Set up and configure an Azure Databricks environment (15–20%)

**Select and configure compute in a workspace**
- [ ] Choose compute type: job, serverless, warehouse, classic, shared
- [ ] Performance settings: CPU, node count, autoscaling, termination, node type, cluster size, pooling
- [ ] Feature settings: Photon, Databricks Runtime/Spark version, ML runtime
- [ ] Install libraries on compute
- [ ] Configure compute access permissions

**Create and organize objects in Unity Catalog**
- [ ] Naming conventions: isolation, dev environment, external sharing
- [ ] Create catalog
- [ ] Create schema
- [ ] Create volumes
- [ ] Create tables, views, materialized views
- [ ] Foreign catalog via connections (Lakehouse Federation)
- [ ] DDL on managed and external tables
- [ ] AI/BI Genie instructions for data discovery

## 2. Secure and govern Unity Catalog objects (15–20%)

**Secure**
- [ ] Grant privileges to principals (user, service principal, group)
- [ ] Table/column-level access control, row-level security
- [ ] Azure Key Vault secrets from Databricks
- [ ] Authenticate data access with service principals
- [ ] Authenticate resource access with managed identities

**Govern**
- [ ] Table/column comments and descriptions for discovery
- [ ] ABAC with tags and policies
- [ ] Row filters and column masks
- [ ] Data retention policies
- [ ] Lineage in Catalog Explorer (owner, history, dependencies, lineage)
- [ ] Audit logging
- [ ] Secure Delta Sharing strategy

## 3. Prepare and process data (30–35%)

**Data modeling in Unity Catalog**
- [ ] Ingestion logic and source config (extraction type, file type)
- [ ] Choose ingestion tool: Lakeflow Connect, notebooks, ADF
- [ ] Batch vs streaming
- [ ] Table format: Parquet, Delta, CSV, JSON, Iceberg
- [ ] Partitioning scheme
- [ ] SCD type
- [ ] Granularity
- [ ] Temporal (history) table
- [ ] Clustering: liquid clustering, Z-ordering, deletion vectors
- [ ] Managed vs unmanaged tables

**Ingest**
- [ ] Lakeflow Connect (batch and streaming)
- [ ] Notebooks (batch and streaming)
- [ ] SQL: CTAS, CREATE OR REPLACE TABLE, COPY INTO
- [ ] CDC feed
- [ ] Spark Structured Streaming
- [ ] Azure Event Hubs streaming
- [ ] Lakeflow Spark Declarative Pipelines with Auto Loader

**Cleanse, transform, load**
- [ ] Profile data with summary stats and distributions
- [ ] Column data types
- [ ] Duplicates, missing values, nulls
- [ ] Filter, group, aggregate
- [ ] Join, union, intersect, except
- [ ] Denormalize, pivot, unpivot
- [ ] Merge, insert, append

**Data quality**
- [ ] Validation: nullability, cardinality, range
- [ ] Data type checks
- [ ] Schema enforcement and schema drift
- [ ] Pipeline expectations (Lakeflow SDP)

## 4. Deploy and maintain data pipelines and workloads (30–35%)

**Design and implement pipelines**
- [ ] Order of operations
- [ ] Notebook vs Lakeflow SDP
- [ ] Task logic for Lakeflow Jobs
- [ ] Error handling in pipelines, notebooks, and jobs
- [ ] Notebook pipeline with precedence constraints
- [ ] Lakeflow SDP pipeline

**Lakeflow Jobs**
- [ ] Create and configure a job
- [ ] Triggers
- [ ] Schedule
- [ ] Alerts
- [ ] Automatic restarts (job or pipeline)

**Dev lifecycle**
- [ ] Git version control best practices
- [ ] Branching, PRs, conflict resolution
- [ ] Testing: unit, integration, E2E, UAT
- [ ] Configure and package Databricks Asset Bundles
- [ ] Deploy bundle via CLI
- [ ] Deploy bundle via REST API

**Monitor, troubleshoot, optimize**
- [ ] Cluster consumption, performance, and cost
- [ ] Lakeflow Jobs: repair, restart, stop, run
- [ ] Spark jobs and notebooks: tuning, bottlenecks, cluster restart
- [ ] Caching, skew, spill, shuffle via DAG, Spark UI, query profile
- [ ] OPTIMIZE and VACUUM
- [ ] Log streaming to Log Analytics (Azure Monitor)
- [ ] Azure Monitor alerts
