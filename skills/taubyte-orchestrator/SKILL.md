---
name: taubyte-orchestrator
description: Master Taubyte workflow skill. Enforces strict order with Dream-by-default routing, scope routing, context logging, and verification.
---

# Taubyte Orchestrator

## When to Use

Use for any Taubyte request before running resource-specific operations.

## Mandatory gate order

1. `taubyte-cli-prereqs` (hard gate: **Node.js** and **Docker** — install/start when possible, else prompt; **`@taubyte/cli`** / **`@taubyte/dream`**; auth via **`taubyte-auth-and-profile`** before GitHub-backed repo work)
2. `taubyte-auth-and-profile` (profiles, `tau login --new`, switches — complements Step 1 login)
3. `taubyte-execution-modes`
4. `taubyte-core-constraints`
5. `taubyte-cloud-selection`
6. `taubyte-scope-routing`
7. `taubyte-project-and-application`
8. `taubyte-context-log`
9. `taubyte-resource-creation`
10. `taubyte-go-sdk-constraints` (Go HTTP handlers: **function** scaffold = root `empty.go` only—never hand-create `lib/` there; **library** WASM = separate skill section + multi-`//export` / `execution.call` wiring)
11. `taubyte-build-runtime-config` (if server-side vars/config/build logic is needed)
12. `taubyte-push-build-verify`
13. `taubyte-dream-local-operations` or `taubyte-remote-cloud-operations`
14. `taubyte-debugging-and-recovery` on failure

## Optional local host access gates (Dream)

- `taubyte-hosts-file-editor` when `.localtau` resources must open by hostname.
- `taubyte-local-host-launch` to validate/launch website or API URLs using FQDN + port.

## Auto-execution policy for local runtime checks

- When user asks to "execute locally", "open URL", "run function", or "test website/function", do this automatically after push/deploy:
  1. run `taubyte-hosts-file-editor` (attempt automatic hosts update first),
  2. run `taubyte-local-host-launch` (hostname-based curl/browser-ready URLs),
  3. return concrete URLs and curl commands.
- If hosts elevation fails, provide exact manual hosts lines and continue verification with `curl --resolve`.
- If Docker is unavailable on Dream path, stop runtime claims and report the blocker explicitly.

## New project hard gate (before imports or resources)

When the user’s request means **create a new Taubyte project** (not “add to the current one”):

1. Run **`taubyte-project-and-application`** → **`tau --defaults --yes new project --name <new>`** then **`tau select project --name <new>`** — **never** skip this because **`tau --json current`** already showed another project.
2. Immediately run **`tau --json current`** and **confirm `Project` equals `<new>`** (exact name). If it does not match, **stop** and fix selection before continuing.
3. **Do not** run **`tau import website`**, **`tau import library`**, **`tau push project`**, or **`tau new` / mutate** domains, databases, functions, etc. until step 2 passes.
4. **Do not** treat an **old workspace folder**, **old `tb_config_*` clone**, or **stale chat context** as the source of truth for which project is active — only **`tau --json current`** after **`tau select project`**.

Log steps 1–2 in **`taubyte-context-log`** (`project_intent=new`, intended name, verified `Project` field).

## Rules

- **New projects:** when the user asks to **create a new project**, follow the **New project hard gate** above and **`taubyte-project-and-application`** → *New project vs reusing an existing project* (**always** `tau new project` + select; do **not** silently reuse the current project unless they said to).
- **Applications:** do **not** create or select an application by default. Follow **`taubyte-scope-routing`** and **`taubyte-project-and-application`**: use **project-only** context for simple **website** or **function** (and global resources) unless the user wants **multiple contexts/applications** or the task is **application-scoped**.
- **Websites:** follow **`taubyte-scope-routing`** → *Website when logically appropriate*. If a browser UI is the natural fit, **plan `tau new website`** (and usually a **Frontend** app in multi-app layouts) — avoid defaulting to API-only for user-facing app briefs unless the user asked for headless / API-only scope.
- **Config notification + sequencing:** ensure `config/config.yaml` has a real `notification.email` (not `""`) before pushing config; push **config first**, wait for the latest **Config build** to succeed, then push **website/library** repos that depend on those config resources (see `taubyte-push-build-verify`).
- **Config notification + sequencing:** ensure `config/config.yaml` has a real `notification.email` (not `""`) before pushing config; push **config first**, wait for the latest **Config build** to succeed, then push **website/library** repos that depend on those config resources (see `taubyte-push-build-verify`).
- Do not skip or reorder gates.
- If `taubyte-cli-prereqs` fails (missing **Node** or **Docker** / daemon not running, missing **`tau login`** when GitHub is needed, or missing `tau`/`dream`), stop immediately.
- Default to non-interactive mode.
- Use JSON verification (`tau --json current`) after context changes.
- Keep commands sequential when selection/context can change.
- Load `taubyte-reference-index` when additional command/topic depth is needed; for **site + `/api/*` + database/library** layouts, use its **Full-stack worked example** (multi-repo, multi-app) instead of assuming project-only scope.
- Load `taubyte-spore-drive-sdk` only for cloud deployment/self-hosting requests.
