---
description: Taubyte critical rules - automation, naming, matcher, git prohibition
alwaysApply: true
---

# Taubyte Core Rules

## QUICK REFERENCE - Critical Rules Checklist

Before ANY action, verify these critical rules:

1. **[MUST]** NEVER prompt user to run commands manually - handle ALL setup automatically
2. **[MUST]** For Dream/Local: NEVER run `tau list projects` (or any tau command that contacts the backend) before ensuring Dream is started AND universe exists. Check `dream status` → start with `dream start` if needed → check universe (e.g. `dream status universe default`) → if universe does not exist, create with `dream new universe default` → only then run `tau list projects` or similar.
3. **[MUST]** Use `dream` commands directly, NEVER use `tau dream`
4. **[MUST]** NEVER create a new universe for each project - always use default universe "default" unless explicitly told otherwise
5. **[MUST]** After creating ANY resource: Push config with `tau push project --config-only`
6. **[MUST]** After code changes: Push code with `tau push project --code-only` then verify build success
7. **[MUST]** Always use `--branch main` for all clone operations
8. **[MUST]** After cloning ANY repo: Verify branch is `main`
9. **[MUST]** NEVER set function `--timeout` to `0` - use valid duration (e.g., `5s`, `30s`, `1m`)
10. **[MUST]** NEVER create resource config files manually - always use `tau new` commands
11. **[MUST]** For Dream deployment: Check Docker status → If not running (Windows: start with `start "" "C:\Program Files\Docker\Docker\Docker Desktop.exe"` and wait for it to load) → Verify running before push-all/push-specific
12. **[MUST]** NEVER use git commands when tau commands fail - investigate and fix root cause instead
13. **[MUST]** After creating ANY resource: Verify config file exists in project config directory
14. **[MUST]** Before creating resources: Verify project is selected with `tau current`
15. **[MUST]** Always use `--template empty` when creating functions or websites
16. **[MUST]** In code: Use matcher (config `match` value) for database/storage SDK calls, never the resource name
17. **[MUST]** For database creation: Always pass `--min`, `--max`, and `--size` (e.g. `--min 1 --max 1 --size 1GB`) when using `--defaults`
18. **[MUST]** HTTP functions: ONE function per (path + method). NEVER use `--method GET,POST,DELETE` or comma-separated methods—config build will fail. Create separate functions (e.g. get_notes, save_note, delete_note) each with a single `--method`.
19. **[MUST]** Dream: After `dream inject push-all --universe [name]` succeeds, run `dream inject push-specific` for **each** website (--rid = numeric `source.github.id`, --fn = `source.github.fullname`). Never skip push-specific for websites.
20. **[MUST]** Code repo layout: Build expects `<app_name>/functions/<function_name>/` at the **root** of the code repo (e.g. `showcase_app/functions/health/`). If `tau new function` creates under `applications/<app_name>/functions/`, move to `<app_name>/functions/` at repo root so the builder finds `.taubyte` and builds succeed.
21. **[MUST]** go-sdk API in code: (1) `Headers().Get` and `Query().Get` return `(string, error)` — use two variables; use `h.Query().Get(...)` only, no `h.URL()`. (2) PubSub `ev.Data()` returns `([]byte, error)` — use two variables. (3) Storage: no `stor.New().File()`; write with `stor.File(name).Add(data, true)`; read with `file.GetFile()` → `*StorageFile` then `sf.Read`/`sf.Close` or `io.Copy`. See `.cursor/sdk-api-must.md`.
22. **[MUST]** Website `.taubyte/build.sh` must never be empty; it must copy or build output into `/out` or the site will not deploy. Static site: `cp index.html /out/` (and optional assets). Node/Vue/React: `npm run build && cp -r dist/* /out`. See `.cursor/taubyte-folder-examples.md` (Website section).

## CORE PRINCIPLES

### [MUST] Automation Rule

- NEVER prompt user to run commands manually
- Handle ALL setup tasks automatically (multiverse, resources, etc.)
- If command runs continuously (e.g., `dream start`), run in background automatically
- NEVER ask user to "please start X" or "please run Y"

### [MUST] Command Requirements

- NEVER use `tau new` commands without ALL required flags - CLI will prompt even with `--yes` if flags missing
- Use `--no-embed-token` or `--no-e` (NOT `--embed-token false`)
- `--yes` only skips confirmation prompts, NOT boolean prompts like embed-token
- For all commands: Use all flags from `.cursor/{resource}-commands.md` except `--tags`
- **[MUST]** For functions and websites: Always use `--template empty` flag - check `--help` to find the template flag
- **[MUST]** For functions: Always include `--language Go` when using `--template empty` to get Go templates (defaults to Rust without this flag)
- **[MUST]** For HTTP functions: Use a single `--method` per function (e.g. `--method GET` or `--method POST`). Never comma-separated methods (e.g. `--method GET,POST,DELETE`)—creates invalid config and config build fails. Create one function per path+method (e.g. get_notes GET, save_note POST, delete_note DELETE).
- **[MUST]** For website creation: NEVER use `--public`. Use `--no-private` for public (non-private) repositories.
- Always start with an empty template named "empty"

### [MUST] Repository Structure

- **Projects**: Two repos - `tb_config_<name>` and `tb_code_<name>` (cloned with `tau clone project`)
- **Websites**: Separate repo `tb_website_<name>` (cloned with `tau clone website --name [name]`)
- **Libraries**: Separate repo `tb_library_<name>` (cloned with `tau clone library --name [name]`)

### [MUST] Code Repo Layout (Application Functions)

- The **build pipeline** looks for function code at `<app_name>/functions/<function_name>/` at the **root** of the code repo (e.g. `Backend/functions/get_leaderboard/`, `showcase_app/functions/health/`).
- It does **NOT** look under `applications/`. So the path must be `showcase_app/functions/event_handler/`, not `applications/showcase_app/functions/event_handler/`.
- If `tau new function` creates under `code/applications/<app_name>/functions/<name>/`, **move** that directory to `code/<app_name>/functions/<name>/` (and remove the empty `applications/` tree) so the builder finds each function's `.taubyte/` and the build succeeds. Otherwise you get: "no taubyte directory found in .../showcase_app/functions/event_handler".

## NAMING CONVENTIONS

### [MUST] Character Restriction Rule

- NEVER use hyphens (`-`) in names for projects, resources (functions, websites, etc.), or universes.
- ALWAYS use underscores (`_`) as the separator.

### [MUST] Database and Storage SDK: Use Matcher, Not Name

- When writing code that calls a database or storage resource, use the **matcher** (match pattern), NOT the resource name.
- **Database:** Call `database.New(match)` where `match` is the `match` value from the database config (e.g. `config/databases/[name].yaml` → use the `match:` field).
- **Storage:** Call `storage.Get(match)` or `storage.New(match)` where `match` is the `match` value from the storage config (e.g. `config/storage/[name].yaml` → use the `match:` field).
- Example: If database config file is `noting14_db.yaml` with `match: notes`, use `database.New("notes")` in code, NOT `database.New("noting14_db")`.
- NEVER use the resource filename or resource name (e.g. `noting14_db`, `my_storage`) in SDK calls; always use the matcher value from config.

### [MUST] Commands Requiring Cloned Project

These commands require project to be cloned locally first: `tau list websites`, `tau list functions`, `tau query builds`, `tau pull project`, `tau push project`

## PROJECT SELECTION VALIDATION

**Before Creating ANY Resource:**

1. Verify project is selected: `tau current`
2. **IF project NOT selected OR wrong project:** `tau select project [name]`, then `tau current` to verify
3. **IF project selection fails:** Check if project exists (`tau list projects`), clone if needed (`tau clone project`), retry selection. For Dream/Local, ensure dream is running and universe exists before calling `tau list projects` (see Dream/Local rules).
4. **VALIDATE:** `tau current` shows correct project before proceeding

**NEVER create resources without verifying project selection first**

## [MUST] Git Prohibition Rule

- NEVER use `git push`, `git commit`, `git add` when `tau push` fails
- NEVER use `git init`, `git remote add` when `tau clone` fails
- NEVER use `git push -u origin main` when `tau push` fails
- NEVER bypass tau commands with direct git operations
- ALWAYS investigate root cause of tau command failure and fix the underlying issue

## DOCUMENTATION ROUTING

- **For commands/flags:** Read `.cursor/{resource}-commands.md` (project, function, website, database, etc.)
- **For logic/architecture:** Read `.cursor/{resource}-logic.md` or `dream-{topic}-logic.md`
- **For SDK usage:** Read `.cursor/sdk-{type}-{topic}.md` (e.g. sdk-function-http.md, sdk-operation-database.md)

## AUTOMATION WORKFLOW DETECTION

**Taubyte context:** Assume when user requests creation of websites with resources, any Taubyte resource, or keywords like "create website", "with database", "taubyte project", "build a", "make a".

**Environment:** "locally", "local", "dream" → Dream/Local; else Remote (default).

**Project:** If project exists/selected, ask "Use existing project '[name]' or create new?" Else generate snake_case name from request.

**GitHub profile:** If username provided, check `tau current`; switch with `tau login <username>`; if profile doesn't exist, create with `tau login --new <username>`.
