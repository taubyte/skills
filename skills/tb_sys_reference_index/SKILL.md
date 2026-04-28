---
name: tb_sys_reference_index
description: Low-token index of deep Taubyte references by topic (commands, Dream, SDK, build/logs, deployment) for selective loading.
---

# Reference Index

Use this skill to load deeper docs only when needed.

## In-repo depth (load these skills)

- **Policy / constraints:** `tb_sys_core_constraints`
- **CLI flags catalog:** `tb_sys_resource_flags_reference`
- **Build / Dream inject / website stacks:** `tb_sys_build_runtime_config`, `tb_wf_dream_inject_and_verify`
- **Push and build polling:** `tb_sys_push_build_verify`, `tb_cmd_tau_query_builds`, `tb_cmd_tau_query_logs`
- **Go SDK surfaces:** `tb_sdk_go_*` (HTTP, PubSub, storage, database, library WASM, local Docker verify)

Optional: add vendored markdown under `<skills_root>/references/` and extend this list.

## Dream/local operations

- Dream readiness and selection: `tb_sys_cloud_selection`
- Inject sequence (Dream build trigger): `tb_wf_dream_inject_and_verify` and `tb_cmd_dream_inject_push_specific`
- Local URL/hosts verification is **explicit-only** and should not run automatically.

## SDK correctness

Use the `tb_sdk_go_*` skills listed above (split by topic).

## Build and logs

Use `tb_sys_push_build_verify`, `tb_sys_build_runtime_config`, and `tb_cmd_tau_query_*` skills.

## Command flags

Canonical non-interactive examples: `tb_sys_resource_flags_reference`. Per-command cards: `tb_cmd_tau_new_*`, `tb_cmd_tau_import_*`, `tb_cmd_tau_push_*`.

## Full-stack worked example (multi-repo, multi-app)

Use when the user wants a **browser app + HTTPS API + persistence**, possibly split across **several GitHub repos** and **two applications** (e.g. Frontend / Backend).

Public reference implementation (**Tower Blocks** + leaderboard):

- **Config** (domains, `applications/*/websites|libraries|databases|functions`): [tb_tower_blocks_game](https://github.com/ZaouiAmine/tb_tower_blocks_game)
- **Website** (Vite + TypeScript + Three.js, `.taubyte` Node build → `/out`): [example-games-tower-blocks-fork](https://github.com/ZaouiAmine/example-games-tower-blocks-fork)
- **Go WASM library** (multiple `//export` handlers, `database.New` path aligned with config): [tb_library_api_leaderboard](https://github.com/ZaouiAmine/tb_library_api_leaderboard)
- **Code** repo (skeleton / layout expected beside config): [tb_code_tower_blocks_game](https://github.com/ZaouiAmine/tb_code_tower_blocks_game)

Implementation map: **`tb_sys_project_application`** (dual-app when needed), **`tb_sys_scope_routing`** (full-stack row), **`tb_sys_resource_creation`** (one library → many functions via `execution.call`), **`tb_sys_build_runtime_config`** (Vite vs `go-wasi` + Dream `push-specific` for website/library), **`tb_sdk_go_*`** (library vs function layout).

## HTTP + database + PubSub + WebSocket (client demo)

Use for **KV persistence**, **channel naming**, **PubSub-triggered functions**, **HTTP publishers** (`pubsub/node`), and **WebSocket URL** handoff to a browser.

- **Config** (`databases/*.yaml`, `messaging/*.yaml`, HTTP + pubsub `functions/*.yaml`): [tb_taubyte_client_demo](https://github.com/ZaouiAmine/tb_taubyte_client_demo)
- **Code** (aligned `database.New`, consumers, `pubsubnode.Channel`, WS bootstrap): [tb_code_taubyte_client_demo](https://github.com/ZaouiAmine/tb_code_taubyte_client_demo)

See **`tb_sdk_go_*`** (Database + PubSub producer sections) and **`tb_sys_core_constraints`** (*Database and messaging* + matcher rules).

## How to use

- Load only the section needed for the current task.
- Prefer index lookup over inlining large rule blocks into active skills.
