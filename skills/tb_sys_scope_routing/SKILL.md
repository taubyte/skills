---
name: tb_sys_scope_routing
description: Routes project-level vs application-scoped work; defaults to a website when a browser UI is logically appropriate; avoids unnecessary applications for simple website/function-only tasks unless needed.
---

# Scope Routing

## Website when logically appropriate (default bias)

When planning a **new project** or **greenfield** work, **include a `website` resource** (explicit repo strategy per **`tb_sys_resource_creation`**) if a **browser UI is a natural part** of the product. Examples: **todo/board/chat/calendar apps**, **CRUD or admin dashboards**, **landing + authenticated app**, **games or interactive demos**, **docs or portal** meant to be opened in a browser, or any brief that says **“app”**, **“UI”**, **“frontend”**, **“SPA”**, or **“site”** without restricting scope to the API.

**Skip the website** only when the project is **clearly non-browser** or the user opts out: **headless API** / **microservice** only, **webhooks**, **PubSub** / **workers** only, **CLI** or **library** deliverable, **pure integration** backend, or the user **explicitly** asks for **API only**, **no UI**, or **backend only**.

If both API and website apply, prefer the **full-stack** row in the scope matrix (often **Frontend** app + **`tau new website`** plus **Backend** app for **`database` / `libraries` / `functions`**). Log **`website_decision`**: `include` | `skip`, with a one-line reason, in **`tb_sys_context_log`**.

## Scope matrix

- **Project / global (default for many tasks):** `project`, `domain`, `website`, `library` — and **standalone function or website work** when the user does **not** ask for multiple applications or named app boundaries.
- **Application-scoped (requires selected application when doing this work):** `application`, `service`, `database`, `storage`, `messaging`, `smartops`.
- **`function`:** treat as **project-level** for **simple, single-function** (or function-only) tasks **without** a named application or multi-context brief; treat as **application-scoped** only when the user ties work to an **application**, or the task is **multi-app / multi-context**, or the **CLI** clearly requires an application for the next commands.
- **Full-stack (website + HTTPS API + database/library):** plan **multiple applications** (common pattern: **Frontend** for the site, **Backend** for **`libraries`**, **`databases`**, and **`functions`** that reference the library). Website YAML lives under the Frontend app; **`database` / library-backed `function`** YAML under Backend. Same **`domains`** reference on site + functions when same-origin **`/api/*`** is desired. See **`tb_sys_reference_index`** → *Full-stack worked example*.

## Rules

0. If the user asked to **create a new Taubyte project**, **`tb_sys_project_application`** must finish (**`tau new project`** + **`tau select project`** + **`tau --json current`** shows the new **`Project`**) **before** **`tb_sys_resource_creation`**, **`tau import`**, or any **`tau new`** resource — otherwise work attaches to the previously selected GitHub project.
1. If the request is **global-only** (website, library, domain, or simple function/site without multi-app), **do not** force application creation or **`tau select application`**.
2. If the request includes **application-scoped** resources (service, database, storage, messaging, smartops, or function **explicitly** under an app), **require** application scope **when** those operations run — create/select an application only per **`tb_sys_project_application`** triggers, not by default.
3. If the request is **mixed**, do **global** resources first (no application unless needed), then **application** resources with **`tau select application`** only for that phase.
4. If ambiguous, ask **one short question** (e.g. single app vs multiple apps vs project-level only).
5. Write the scope decision and **reason** (including **“no application”**) to **`tb_sys_context_log`**.
6. Apply **Website when logically appropriate**: do not ship **API-only** for user-facing “app” briefs when a site is the natural delivery surface; skip the website only for headless / explicit API-only scope (see section above).
