---
description: Doc routing - .cursor commands, logic, SDK files
alwaysApply: false
---

# Taubyte Reference

## COMMAND REFERENCE

Use `.cursor/{resource}-commands.md` for full command syntax. Key resources: project, function, website, library, domain, database, storage, messaging.

## DOCUMENTATION INDEX

**CLI Commands:** `.cursor/{resource}-commands.md` — auth, project, function, application, website, library, domain, database, storage, messaging, smartops, service, network, builds-logs, context, cli-utilities, config, fixture, repo, multiverse, universe, simple-node, dream-env, testing, debugging

**Logic/Architecture:** `.cursor/{resource}-logic.md` — same resources plus dream-multiverse, dream-universe, dream-services, dream-fixtures, dream-api, dream-simple-node

**SDK:** `.cursor/sdk-{type}-{topic}.md` — sdk-function-http/pubsub/p2p, sdk-operation-database/storage/messaging/http-client/http/ipfs/p2p, sdk-pattern-errors/events. Database/storage: use **matcher** from config, not resource name. **Must-use API (avoid compile errors):** `.cursor/sdk-api-must.md` — Headers/Query/PubSub two return values, storage GetFile/Add, no h.URL().

**Additional:** `.cursor/taubyte-folder-examples.md` for .taubyte structure. **Build/test functions locally:** `.cursor/build-test-functions-docker.md` — Docker one-shot build per function to catch compile errors before push. **Check logs and debug failed builds:** `.cursor/check-logs-debug.md` — `tau query builds`, `tau query logs <id>`, what to look for, and troubleshooting links.

**Dream hosts (user):** `.cursor/dream-hosts-instructions.md` — step-by-step instructions to edit the hosts file so you can open Dream app/API URLs in the browser (Windows + Linux/macOS).

**Deploy Taubyte cloud (Spore Drive):** Use the **spore-drive-sdk** skill when the user wants to deploy a Taubyte cloud (e.g. "deploy a cloud on domain X, subdomain Y, on these servers"). Do **not** use a premade spore-drive; create deployment code using `@taubyte/spore-drive` SDK. Extract from the user's message: domain, subdomain, servers (hostname, public IP, optionally location). Prompt only for what's missing (SSH key, etc.). Use SSH user ubuntu first, then root if that fails; ask user only if both fail — you are **obliged** to ask rather than fail. Create a deploy project (e.g. `spore-drive-deploy/`), write the code, run `npm install` and `npm run displace`; do everything in the background. See `.cursor/taubyte/spore-drive-cloud-deployment.md` for required inputs and DNS instructions.

## ROUTING RULES

- Deploy taubyte cloud / spore drive / self-host taubyte → `spore-drive-cloud-deployment.md`
- CLI command (tau/dream) → `{topic}-commands.md`
- Go code/API → `{topic}-logic.md` or `dream-{topic}-logic.md`
- Architecture/concepts → `{topic}-logic.md`
- Dream Go API → `dream-{topic}-logic.md`
- Dream CLI → `dream-env-commands.md`
- SDK, function impl, resource ops → `sdk-{type}-{topic}.md`
