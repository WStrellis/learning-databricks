# Databricks Learning Project

One project for several Databricks courses and certs. Each course has its own folder under `courses/`. The user works with Azure Databricks day to day and has passed 3 other Azure certs.

## Courses

| Order | Folder | Course | Status |
|---|---|---|---|
| 1 | `courses/azure-platform-architect/` | Databricks Academy: Azure Databricks Platform Architect | Active |
| 2 | `courses/dp-750/` | Microsoft DP-750: Implementing Data Engineering Solutions Using Azure Databricks | Paused (LP1-M1 done) |

Each course's `README.md` has its sources, outline/checklist, note naming, and progress. **Read the active course's README before starting work.** If the user doesn't say which course, assume the Active one.

## Working agreement (all courses)

- **Claude takes the notes** for each module → `courses/<course>/notes/`.
- **Claude quizzes** the user. Use scenario style like the real exams (pick the best option, "which TWO", order the steps), not trivia only. Log results in `courses/<course>/quizzes/`.
- **The user does the labs themselves.** Later Claude designs new labs (`courses/<course>/labs/`) and **only checks** the user's work (read-only Azure CLI / Databricks CLI / MCP). **Never create, modify, or fix resources for them.** Give hints first, then the answer only if they ask.
- Flag where course content and the current docs disagree.
- **Cross-course links:** courses overlap a lot (workspaces, Unity Catalog, compute, networking). When a topic is already covered in another course's notes, link to that file instead of rewriting it, and note only what's new or different.

## Layout

```
CLAUDE.md                  shared rules + course index (this file)
courses/<course>/
  README.md                sources, outline/checklist, note naming, progress
  notes/                   one notes file per module
  quizzes/                 quiz logs and weak spots
  labs/                    Claude-designed practice labs
```

To add a course: create `courses/<slug>/` with the same structure and add a row to the table above.

## Shared sources

- Azure Databricks docs: https://learn.microsoft.com/en-us/azure/databricks/
- Databricks docs: https://docs.databricks.com/
