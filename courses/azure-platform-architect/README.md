# Azure Databricks Platform Architect (Databricks Academy)

E-learning course, English, about 6 hours. 3 sections, 25 lessons. Ends with a short quiz; passing it earns a Databricks badge (no separate proctored exam).

## Sources

- Course: https://customer-academy.databricks.com/learn/courses/7541/azure-databricks-platform-architect (needs a Databricks Academy login)
- Badge / certification page: https://www.databricks.com/learn/certification/databricks-on-azure
- Docs: https://learn.microsoft.com/en-us/azure/databricks/

## Outline

Read from the course syllabus on 2026-11-21. Lessons are SCORM modules unless marked otherwise.

**Intro**
- Before we get started (slides)
- Course Logistics Review - General Information (HTML)

**Section 1: Azure Databricks Workspace Administration Fundamentals**
1. Azure Databricks Service Architecture
2. Workspace Types and Tiers
3. Deploying Workspaces
4. Account Administration on Azure
5. Unity Catalog Setup on Azure
6. Operating Azure Databricks Estates
7. Demo: Azure Databricks Workspace Administration Fundamentals (video)

**Section 2: Azure Databricks Cloud Integrations**
1. Microsoft Entra ID Native Integration
2. Automatic Identity Management
3. Service Principals, Managed Identities, and API Auth
4. Storage and Key Vault Integration
5. Integrating Azure Services
6. Monitoring, Policy, and Cost Integrations
7. Demo: Azure Databricks Cloud Integrations (video)

**Section 3: Azure Databricks Networking and Security Fundamentals**
1. Managed VNet vs VNet Injection
2. Egress Control with UDRs and Firewalls
3. Azure Private Link
4. Serverless Networking with NCCs
5. Customer-Managed Keys
6. Perimeter and Compliance
7. Demo: Azure Databricks Networking and Security Fundamentals (video)

**Wrap-up**
- Summary and Next Steps (HTML)
- Quiz - Azure Databricks Platform Architect (test)

## Conventions

- Notes: `notes/S<n>-<slug>.md`, one per section (for example `notes/S1-workspace-administration.md`). The section demo goes in the same file.
- Overlaps with DP-750 (link instead of repeating):
  - S1 Service Architecture, Workspace Types and Tiers, Deploying Workspaces → `../dp-750/notes/LP1-M1-explore-azure-databricks.md`
  - S1 Unity Catalog Setup, S2 Entra ID / service principals / Key Vault → DP-750 governance and security topics (see `../dp-750/exam-outline.md`)

## Progress

- [ ] Intro
- [ ] S1 Workspace Administration Fundamentals (notes done: 1.1–1.6)
- [ ] S2 Cloud Integrations (notes done: 2.1–2.2)
- [ ] S3 Networking and Security Fundamentals
- [ ] Summary and course quiz (earns the badge)
