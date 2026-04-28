---
name: tb_sys_resource_creation
description: Scope-aware resource creation workflow. Uses non-interactive mode by default and references the shared flags catalog.
---

# Resource Creation

## Preconditions

- `tb_sys_core_constraints` loaded
- `tb_sys_scope_routing` completed
- `tb_sys_project_application` completed for **project** (and **application** only if the task required it — do not assume an application exists)
- **`tau --defaults --yes json current`** was run after the last **`tau select project`** / **`tau new project`**; the **`Project`** field matches the task’s intended project (see **`tb_sys_router`** → *New project hard gate* — **never** import or create resources under a stale selection). If the user named a project by name and **Project** differs, run **`tau --defaults --yes select project --name <that>`** first, then re-verify.
- `tb_sys_context_log` initialized
- `tb_sys_build_runtime_config` available for server-side config/build logic

## Order

- **Global / project-level** resources first (**`domain`**, and **`website` / `library`** only when **`tb_sys_scope_routing`** keeps them at project scope) — **no** application step unless routing says otherwise.
- **Full-stack multi-app** (see **`tb_sys_reference_index`** → *Full-stack worked example*): **`domain`** at project level → create/select **Frontend** → **website** there → create/select **Backend** → **`library`**, **`database`**, **`messaging`** (when using PubSub/WebSocket), **`functions`** there. Create **`messaging`** before or together with functions that publish or subscribe to that channel; create **`library`** / **`database`** before or in lockstep with function YAML that references them. Push **project config** and wait for config build success before relying on realtime or new routes.
- **Website default:** when **`tb_sys_scope_routing`** → *Website when logically appropriate* applies, **`tau new website`** (and import/push per below) is **in scope** for that project — not optional for user-facing app briefs unless the user chose headless / API-only.
- **Application-scoped** resources next (**database**, **storage**, **messaging**, **service**, **smartops**, and **function** when scoped to an application) — run **`tau select application`** only for the phase that needs it; **do not** create an application for website-only or simple global function work.

## Rules

- Use default non-interactive mode unless user requested interactive.
- For full flags, use `tb_sys_resource_flags_reference`.
- Use `MSYS_NO_PATHCONV=1` for path-like flags in Git Bash.
- **Never** use **`--generated-fqdn-prefix`**, **`--g-prefix`**, or any synonym on **`tau new domain`** (unconditional ban — see **`tb_sys_resource_flags_reference`**).
- If server-side config is needed, update `.taubyte/config.yaml`; if **env vars** are needed, declare them in **`.taubyte/build.sh`** only (see `tb_sys_build_runtime_config`). Validate both before push.
- For Go function implementation correctness, apply `tb_sdk_go_*`.
- After creating a serverless function with `--template empty`, immediately edit **`empty.go` at the function root** (same directory as `go.mod` — never under `lib/`) with a real implementation. **Do not** create `lib/`, move `empty.go` into `lib/`, or add hand-written **`main.go`**; see **`tb_sdk_go_*`** (layout + Docker WASM test).
- Website/library creation must use explicit repo strategy; never rely on implicit repo auto-selection.
- Deterministic default for generated repos:
  - website: `tb_code_<project>_<website>`
  - library: `tb_code_<project>_<library>`
- Immediately import after creation to sync local repository clone (**only after** preconditions above — especially correct **`Project`** in **`tau --json current`**):
  - `tau import website --name <site>`
  - `tau import library --name <lib>`
- Immediately verify bound repo fullname/ID in config yaml after creation/import; if mismatch, stop and fix before any push.
- Update context log after every mutation.

---

## One Go WASM library, multiple HTTP functions

When several routes share one build:

1. Create **one** library resource + repo (**`libraries/<name>.yaml`**, GitHub source).
2. Add **separate** **`functions/<handler>.yaml`** files under the **same Backend (or chosen) application**, each with **`source: libraries/<sameLibraryName>`** (path relative to that application).
3. Split behavior with **`trigger.type: https`** (or **`http`** per CLI), **`trigger.method`**, **`trigger.paths`**, and **`execution.call`** — each **`call`** must match a **`//export <call>`** symbol in the library sources. Pure PubSub consumers may omit **`domains:`**; HTTP routes keep **`domains: [...]`** as usual.
4. Define a **`database`** resource whose **`match`** (e.g. **`/leaderboard`** or **`appdata`**) is **identical** to the string passed to **`database.New(...)`** in the Go code (including a leading **`/`** when the YAML uses one). After editing YAML or Go, grep for **`database.New(`** and confirm every literal matches some **`databases/*.yaml`** in the **same application**.
5. For **`libraries/*.yaml`**, set **`source.path`** to the path **inside the library repository** where sources live (must match the cloned repo layout — not an invented folder). Mismatch is a common cause of “API runs but data doesn’t persist” or wrong handlers.

Verify each resource’s bound **`github.fullname`** after import/create. Multi-repo splits (config vs website vs library vs code skeleton) follow the same explicit-repo rules as any **`tau import website`** / **`tau import library`** workflow.

For a concrete YAML + repo layout, see **`tb_sys_reference_index`** → *Full-stack worked example*.
