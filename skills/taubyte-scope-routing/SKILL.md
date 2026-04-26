---
name: taubyte-scope-routing
description: Routes project-level vs application-scoped work; avoids creating or selecting an application for simple website/function-only tasks unless needed.
---

# Scope Routing

## Scope matrix

- **Project / global (default for many tasks):** `project`, `domain`, `website`, `library` — and **standalone function or website work** when the user does **not** ask for multiple applications or named app boundaries.
- **Application-scoped (requires selected application when doing this work):** `application`, `service`, `database`, `storage`, `messaging`, `smartops`.
- **`function`:** treat as **project-level** for **simple, single-function** (or function-only) tasks **without** a named application or multi-context brief; treat as **application-scoped** only when the user ties work to an **application**, or the task is **multi-app / multi-context**, or the **CLI** clearly requires an application for the next commands.

## Rules

1. If the request is **global-only** (website, library, domain, or simple function/site without multi-app), **do not** force application creation or **`tau select application`**.
2. If the request includes **application-scoped** resources (service, database, storage, messaging, smartops, or function **explicitly** under an app), **require** application scope **when** those operations run — create/select an application only per **`taubyte-project-and-application`** triggers, not by default.
3. If the request is **mixed**, do **global** resources first (no application unless needed), then **application** resources with **`tau select application`** only for that phase.
4. If ambiguous, ask **one short question** (e.g. single app vs multiple apps vs project-level only).
5. Write the scope decision and **reason** (including **“no application”**) to **`taubyte-context-log`**.
