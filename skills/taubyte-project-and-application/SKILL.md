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

(Use **`tau select project`** alone if the project already exists.)

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
