---
name: taubyte-context-log
description: Auto-maintains a context log file inside the current project (cloud, scope, selected resources, latest actions) to reduce AI hallucinations.
---

# Context Log (Mandatory)

## Rule

For every Taubyte workflow, the agent must create/update a log file inside the current project automatically.

## Log file location

- Preferred: `<project_root>/.taubyte_ai/context.log.md`
- If `.taubyte_ai` does not exist, create it.
- Also maintain append-only: `<project_root>/.taubyte_ai/logs.txt`

## Minimum required log content

1. Timestamp
2. Cloud type and cloud target (Dream universe or remote FQDN)
3. Current profile, project, and application (or global scope)
4. Scope decision: global/application + short reason
5. Environment readiness state
   - Dream status (ready/not ready)
   - Docker status (ready/not ready, if Dream path)
6. Known resources by type (domains, databases, storage, messaging, services, functions, websites, libraries)
7. Last commands executed and result summary
8. Build tracking
   - latest relevant build/job IDs
   - last log diagnosis (root cause summary)
9. CLI versions snapshot
   - `tau version`
   - `dream --version || dream --help` (some Dream builds do not expose `--version`)

## Update triggers

- After login/profile change
- After cloud selection
- After Dream/Docker readiness checks
- After project/application selection
- After scope routing decision
- After resource creation/edit/delete
- After push/build/log checks
- After any recovery action
- After any CLI install/reinstall attempt

## logs.txt format (lightweight but complete)

Append new entries after context updates using this compact structure:

```txt
[timestamp] stage=<phase> status=<ok|error>
context: profile=<...> cloud=<...> project=<...> app=<...>
tau_version: <value or error>
dream_version: <value or error>
error: <short message or none>
fix: <short action taken>
progress: <short current state>
footprint: files=<count/list> resources=<count/list>
```

Rules:
- Keep each entry short; include only actionable details.
- Always include error + fix when a command fails.
- Never skip version capture in logs.

## Data collection commands (example)

```bash
tau --json current
tau --json list domains
tau --json list databases
tau --json list storages
tau --json list messaging
tau --json list services
tau --json list functions
tau --json list websites
tau --json list libraries
tau query builds --since 1h
```
