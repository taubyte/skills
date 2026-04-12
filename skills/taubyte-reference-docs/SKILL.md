---
name: taubyte-reference-docs
description: Tau CLI command references, resource logic, Dream, SDK operations, and Taubyte debugging docs. Use when implementing APIs, websites, databases, or troubleshooting builds.
---

# Taubyte reference documentation

This skill bundles the Taubyte **command** and **logic** markdown references, SDK operation guides, and debugging notes as plain files under `references/`.

## How to use

1. For **CLI flags** on a resource, open the matching `*-commands.md` (e.g. `function-commands.md`, `database-commands.md`).
2. For **behavior and architecture**, open the matching `*-logic.md`.
3. For **Go SDK** usage, start with `sdk-api-must.md`, then `sdk-function-http.md`, `sdk-operation-database.md`, `sdk-operation-storage.md`, etc.
4. For **Dream/local** workflows, use `dream-*` and `dream-env-commands.md` / `dream-env-logic.md` as needed.
5. For **websites** build output, see `taubyte-folder-examples.md`.

## Key entry points

| Topic | Files |
|-------|--------|
| Project / clone / push | `project-commands.md`, `project-logic.md`, `repo-commands.md`, `repo-logic.md` |
| Functions / HTTP | `function-commands.md`, `function-logic.md`, `sdk-function-http.md`, `sdk-function-pubsub.md` |
| Databases / storage | `database-commands.md`, `database-logic.md`, `storage-commands.md`, `storage-logic.md`, `sdk-operation-database.md`, `sdk-operation-storage.md` |
| Websites | `website-commands.md`, `website-logic.md`, `taubyte-folder-examples.md` |
| Dream / local | `dream-env-commands.md`, `dream-multiverse-logic.md`, `dream-universe-logic.md`, … |
| Builds / logs | `builds-logs-commands.md`, `build-test-functions-docker.md` |
| Spore Drive (inputs, DNS) | `spore-drive-cloud-deployment.md` (also linked from **spore-drive-sdk** skill) |

All files are in `references/` with the same filenames as the source `.cursor` docs.
