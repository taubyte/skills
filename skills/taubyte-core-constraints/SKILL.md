---
name: taubyte-core-constraints
description: Non-negotiable Taubyte constraints that prevent config/build/runtime failures.
---

# Core Constraints

## Must-follow rules

1. Never ask user to run setup commands manually; agent performs setup.
2. Run CLI preflight (`taubyte-cli-prereqs`) before all Taubyte operations.
3. For Dream/local, never run backend-contacting `tau` commands before Dream is up and universe exists.
4. Use `dream` commands directly; never use `tau dream`.
5. Use default universe `default` unless user explicitly requests another.
6. After resource creation, push config (`tau push project --config-only`) before relying on resource visibility.
7. After code changes, push code (`tau push project --code-only`) and verify builds/logs.
8. Never set function timeout to `0`; use valid durations (`30s`, `1m`, etc.).
9. HTTP functions: one function per path+method; never comma-separated methods.
10. For functions/websites in automation, use empty template; for Go functions, include `--language Go`.
11. Use matcher values for DB/storage SDK calls, not human-facing resource names or YAML filenames (`description` / basename are not runtime keys). The Go string must match YAML **`match`** (and regex pattern when `useRegex: true`). Examples: `match: appdata` ↔ `database.New("appdata")`; `match: /todos` ↔ `database.New("/todos")` — leading slash must match exactly.
12. Do not bypass failing `tau push project` with direct git operations.
13. On Git Bash Windows for path-like flags, prefix with `MSYS_NO_PATHCONV=1`.
14. Website `.taubyte/build.sh` must be non-empty and must write deploy output to `/out`.
15. Verify project selection with `tau --json current` before resource mutations.
16. Website/library git fallback is an exception path only when `tau push website|library` is unavailable; never apply this exception to project config/code pushes.
17. Never create website/library without explicit repository strategy:
    - use `--generate-repository` with deterministic `--repository-name`, or
    - use explicit existing `--repository-name`/`--repository-id`.
18. After creating website/library, verify repository binding in config (`websites/*.yaml` or `libraries/*.yaml`) before pushing.
19. **Go functions:** **`empty.go` stays at the function root** next to `go.mod`. **Never** manually create **`lib/`**, never move `empty.go` under `lib/`, and **never** hand-author **`main.go`** in the function folder to fix builds. For local WASM verification, use the **`taubyte/go-wasi`** Docker recipe in **`taubyte-go-sdk-constraints`** — not `lib/` + `main.go` shims.
20. **New project:** If the user asked to **create a new project**, run **`tau new project`** + **`tau select project`**, then **`tau --json current`** and confirm **`Project`** matches the new name **before** any **`tau import`**, **`tau push project`**, or **`tau new`** resource command. Stale selection is the usual cause of “imports pulling the wrong GitHub project.”
21. **Project config notification email:** ensure `config/config.yaml` has a valid `notification.email` (not `""`). Prefer `git config user.email`; if unavailable, ask the user for the email. Write it as a plain YAML value (no quotes) to avoid `mail: no address` config-compiler failures.
22. **Push ordering:** always push **project config** first and **wait for the latest Config build to succeed** before pushing **website/library** code that depends on those resources being present in config.
23. **HTTP handlers with a body:** set headers (e.g. `Content-Type`) as needed, **write the body** (`Write` / equivalent), then **`Return` with the final status** as the **last** step. Returning before writing yields empty or broken JSON and clients that look like “API does nothing.”
24. After adding or renaming **databases** or **messaging**, grep the codebase for `database.New(` and `pubsubnode.Channel(` (or equivalent) and diff literals against `config/applications/<app>/databases/*.yaml` and `messaging/*.yaml` before declaring the task done.

## Database and messaging (config ↔ code)

- **Database:** path is `config/applications/<app>/databases/<name>.yaml`. Every `database.New(...)` in WASM for that app must use the same string as resource **`match`** when `useRegex: false`. Typical YAML also includes `useRegex`, `access.network`, `storage.size`, `replicas.min` / `replicas.max` — follow CLI-generated shapes when unsure.
- **Messaging + PubSub:** the **messaging** resource defines `channel.match` (and `bridges.mqtt` / `bridges.websocket`). PubSub-triggered functions use `trigger.type: pubsub` and **`trigger.channel`** equal to that match. Producers in HTTP code use the same channel string with **`pubsub/node`**. When both messaging and function YAML include **`local`**, keep them consistent. Enable **`websocket`** in messaging config if handlers expose `WebSocket().Url()` to browsers.
- **Debugging “database open failed”:** first compare YAML **`match`** vs every `database.New` literal in that application (including `/`).

## Layout constraints

- Project has `tb_config_<name>` + `tb_code_<name>`.
- Website/library have separate repositories.
- Function build layout must be discoverable by build system (avoid misplaced function paths). For Go serverless functions, that means **root `empty.go` only** for authored sources; do not invent a `lib/` tree or commit ad-hoc `main.go` beside it.
