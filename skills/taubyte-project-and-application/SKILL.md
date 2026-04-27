---
name: taubyte-project-and-application
description: Creates/selects project; creates or selects an application only when scope or task requires it—not for simple website/function-only work.
---

# Project and application

## Default (important)

**Do not** create an application “just because” you created or selected a project.

- After **`tau select project`**, **stop** unless **`taubyte-scope-routing`** (and the user’s request) show that an **application** is actually needed.
- **Never** run **`tau new application`** unless one of the **“Create or select an application”** triggers below is true.

Log the decision (including **“no application; project-level only”**) in **`taubyte-context-log`**.

---

## Website vs API-only (greenfield)

Follow **`taubyte-scope-routing`** → *Website when logically appropriate*. For **new** projects where end users would reasonably open a **browser**, default to **including a website** (and the **Frontend + Backend** split from *Multi-application pattern* when the backend uses **`database`**, **`libraries`**, or multi-route **`functions`**). Do **not** add a website when the brief or user is **explicitly** API-only / headless / no UI.

---

## New project vs reusing an existing project

When the user asks to **create a new project** (or equivalent wording: “new Taubyte project”, “start a project from scratch”, “greenfield project”, etc.):

1. **Pick a distinct `--name`** (avoid reusing an existing project name unless the user explicitly wants that repo — collisions confuse humans and tools).
2. **Always** run **`tau --defaults --yes new project --name <name> ...`**, then **`tau select project --name <name>`**.
3. **Mandatory verification:** run **`tau --json current`** and assert **`Project`** is **exactly** `<name>`. If not, **stop** — do not import, push, or create resources until selection is fixed.
4. **Mandatory project email:** open `config/config.yaml` and ensure `notification.email` is a real email address (not `""`). Prefer `git config user.email`. Write it as a plain YAML value (no quotes) to avoid `mail: no address` config compiler failures.
5. **Forbidden until step 3 passes:** **`tau import website`**, **`tau import library`**, **`tau push project`**, **`tau new domain`**, **`tau new database`**, **`tau new function`**, **`tau new website`**, **`tau new library`**, or any other mutation tied to a project. Those operations attach to **whatever project is currently selected**; a stale selection is what makes it look like “GitHub keeps importing the old project.”
6. **Do not** reuse **`tau --json current`**’s existing **Project** or assume the open editor folder is the active project **unless** the user clearly said to extend that project (by name, “current project”, “add to skills-test”, etc.). **Do not** skip **`tau new project`** just because a project was already selected in a previous chat turn.

If the brief is ambiguous (“build a todo app”) and a project is already selected, **ask one short question** (new dedicated project vs extend current) before mutating resources.

Log in **`taubyte-context-log`**: **`project_intent`** (`new` vs `extend`), **project name**, **`verified_project_after_select`** (yes/no + JSON **`Project`** value), and whether **`tau new project`** was run vs **`tau select project` only**.

---

## When **not** to create or select an application

Stay at **project** context (no **`tau new application`**, no **`tau select application`**) when the work is any of:

- **Website** only (create/import/push/configure site).
- **Library** only.
- **Domain** (or other **global** resources) only.
- **A single function** or small function-only task **without** the user asking for separate apps, “multiple contexts,” or an explicit application boundary.
- The user did **not** name an application and did **not** describe **multiple applications** or **multi-context** layouts.

In those cases, complete the task under **project** scope unless the **CLI** returns an error that clearly requires **`tau select application`**—then reassess with the user rather than inventing a new app.

---

## When **to** create or select an application

Do **`tau new application`** / **`tau select application`** only when at least one of:

1. The user **explicitly** wants one or more **named applications** or a **multi-app / multi-context** project.
2. **`taubyte-scope-routing`** marks the task as **application-scoped** (e.g. work that assumes an application context: **service**, **database**, **storage**, **messaging**, **smartops**, or a **function** explicitly tied to a chosen/named application).
3. You are about to run commands that **require** a selected application per **`tau --help`** / error output for that command.

If an application **already exists** and the user named it, use **`tau select application --name <app>`** only—do **not** create a duplicate with **`new application`**.

---

## Multi-application pattern (Frontend + Backend)

Use when the brief needs **all** of: a **website** on the project domain, **HTTP functions** (often backed by a **Go WASM library**), and **application-scoped persistence** (**`database`**, etc.). This is an explicit **multi-context** layout — not the default project-only website path. For user-facing **apps** with persistence or a non-trivial API, treat this as the **default** layout unless **`taubyte-scope-routing`** says to skip the website (headless / API-only).

Typical split (names vary):

1. **Frontend application** — **`websites/*.yaml`** sourcing the **front-end GitHub repo**; **`domains`** aligned with the API so the SPA can use **same-origin** **`fetch("/api/...")`** when no separate API base is configured (see **`taubyte-build-runtime-config`**).
2. **Backend application** — **`libraries/*.yaml`** (library WASM repo), **`databases/*.yaml`** with **`match`** aligned to **`database.New("/...")`** in code, **`functions/*.yaml`** with **`source: libraries/<library>`** and per-route **`execution.call`** matching **`//export <name>`** in the library (see **`taubyte-resource-creation`**).

Run **`tau select application`** when mutating resources under each app; log which application is active in **`taubyte-context-log`**.

---

## Steps

### 1. Verify context

```bash
tau --json current
```

### 2. Create or select **project** only

```bash
tau --defaults --yes new project --name <project> --description "<desc>" --private --no-embed-token
tau select project --name <project>
```

Use **`tau select project --name <existing>`** alone only when extending an **existing** project the user chose—**not** when they asked for a **new** project (see **“New project vs reusing an existing project”** above).

### 3. Application — **only if triggers above match**

```bash
tau --defaults --yes new application --name <app> --description "<desc>"
tau select application --name <app>
```

Skip this block entirely when **“When not to create”** applies.

### 4. Verify

```bash
tau --json current
```

Record **project** and, if applicable, **application** (or **none / global-only**) in the context log.

If the user asked for a **new** project in this task, **re-read** the **`Project`** field and confirm it matches the name from step 2 before leaving this skill.
