---
name: taubyte-reference-index
description: Low-token index of deep Taubyte references by topic (commands, Dream, SDK, build/logs, deployment) for selective loading.
---

# Reference Index

Use this skill to load deeper docs only when needed.

## Core rules and policy

- `skills-main/skills/taubyte-core-rules/references/taubyte-core.md`
- `skills-main/skills/taubyte-core-rules/references/taubyte-error-handling.md`
- `skills-main/skills/taubyte-core-rules/references/taubyte-deployment.md`
- `skills-main/skills/taubyte-core-rules/references/taubyte-resources.md`

## Dream/local operations

- `skills-main/skills/taubyte-core-rules/references/taubyte-dream.md`
- `skills-main/skills/taubyte-reference-docs/references/dream-env-commands.md`
- `skills-main/skills/taubyte-reference-docs/references/dream-hosts-instructions.md`

## SDK correctness

- `skills-main/skills/taubyte-reference-docs/references/sdk-api-must.md`
- `skills-main/skills/taubyte-reference-docs/references/sdk-function-http.md`
- `skills-main/skills/taubyte-reference-docs/references/sdk-function-pubsub.md`
- `skills-main/skills/taubyte-reference-docs/references/sdk-operation-storage.md`

## Build and logs

- `skills-main/skills/taubyte-reference-docs/references/build-test-functions-docker.md`
- `skills-main/skills/taubyte-reference-docs/references/check-logs-debug.md`
- `skills-main/skills/taubyte-reference-docs/references/builds-logs-commands.md`
- `skills-main/skills/taubyte-reference-docs/references/taubyte-folder-examples.md`

## Command flags

- `skills-main/skills/taubyte-reference-docs/references/command-flags-reference.md`
- `skills-main/skills/taubyte-reference-docs/references/project-commands.md`
- `skills-main/skills/taubyte-reference-docs/references/function-commands.md`
- `skills-main/skills/taubyte-reference-docs/references/website-commands.md`
- `skills-main/skills/taubyte-reference-docs/references/library-commands.md`

## Full-stack worked example (multi-repo, multi-app)

Use when the user wants a **browser app + HTTPS API + persistence**, possibly split across **several GitHub repos** and **two applications** (e.g. Frontend / Backend).

Public reference implementation (**Tower Blocks** + leaderboard):

- **Config** (domains, `applications/*/websites|libraries|databases|functions`): [tb_tower_blocks_game](https://github.com/ZaouiAmine/tb_tower_blocks_game)
- **Website** (Vite + TypeScript + Three.js, `.taubyte` Node build → `/out`): [example-games-tower-blocks-fork](https://github.com/ZaouiAmine/example-games-tower-blocks-fork)
- **Go WASM library** (multiple `//export` handlers, `database.New` path aligned with config): [tb_library_api_leaderboard](https://github.com/ZaouiAmine/tb_library_api_leaderboard)
- **Code** repo (skeleton / layout expected beside config): [tb_code_tower_blocks_game](https://github.com/ZaouiAmine/tb_code_tower_blocks_game)

Implementation map: **`taubyte-project-and-application`** (dual-app when needed), **`taubyte-scope-routing`** (full-stack row), **`taubyte-resource-creation`** (one library → many functions via `execution.call`), **`taubyte-build-runtime-config`** (Vite vs `go-wasi` + Dream `push-specific` for website/library), **`taubyte-go-sdk-constraints`** (library vs function layout).

## HTTP + database + PubSub + WebSocket (client demo)

Use for **KV persistence**, **channel naming**, **PubSub-triggered functions**, **HTTP publishers** (`pubsub/node`), and **WebSocket URL** handoff to a browser.

- **Config** (`databases/*.yaml`, `messaging/*.yaml`, HTTP + pubsub `functions/*.yaml`): [tb_taubyte_client_demo](https://github.com/ZaouiAmine/tb_taubyte_client_demo)
- **Code** (aligned `database.New`, consumers, `pubsubnode.Channel`, WS bootstrap): [tb_code_taubyte_client_demo](https://github.com/ZaouiAmine/tb_code_taubyte_client_demo)

See **`taubyte-go-sdk-constraints`** (Database + PubSub producer sections) and **`taubyte-core-constraints`** (*Database and messaging* + matcher rules).

## How to use

- Load only the section needed for the current task.
- Prefer index lookup over inlining large rule blocks into active skills.
