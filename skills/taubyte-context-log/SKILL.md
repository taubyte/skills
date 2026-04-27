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
   - build/job completion status (queued/running/success/failed)
   - wait/poll notes (how long waited, last observed state)
9. CLI versions snapshot
   - `tau version`
   - `dream --version || dream --help` (some Dream builds do not expose `--version`)
10. **Project intent** (required when the user said *new project* or *extend existing*):
    - `project_intent`: `new` | `extend`
    - `intended_project_name` (for `new`, the name passed to **`tau new project`** / **`tau select project`**)
    - `verified_project_after_select`: copy the **`Project`** value from **`tau --json current`** immediately after selection, and note whether it matches `intended_project_name`
11. **`website_decision`** (when **`taubyte-scope-routing`** applies): `include` | `skip` plus a one-line reason — see *Website when logically appropriate* in that skill.
12. **`notification_email`**: the value of `notification.email` from `config/config.yaml` (must be a real email, not `""`).
12. **`notification_email`**: the value of `notification.email` from `config/config.yaml` (must be a real email, not `""`).

## Build job waiting policy (do not race ahead)

Builds are often **asynchronous** and can take minutes. After any step that triggers a build (push, inject, retry, build command), **do not continue** to downstream steps that assume a ready build until you have **observed completion**.

Rules:

- If the workflow triggers a build, you must **wait/poll** until the relevant build(s) are **success** or **failed**, or until a reasonable timeout is reached.
- Prefer **short polling with backoff** (e.g. every 5–10s initially, then 15–30s) and record the latest status in `context.log.md` and `logs.txt`.
- If the build is still queued/running at timeout: **stop**, tell the user it is still running, and capture the last known build/job IDs and status so the next run can resume.

Recommended commands (examples; use the project’s actual build/log commands):

```bash
# See recent builds and their statuses (capture IDs)
tau query builds --since 1h

# Re-run until completion state changes (poll)
tau query builds --since 1h
```

When a build fails, capture the build/job ID(s), then pull logs/diagnosis (per the relevant push/build/verify skill) and record:
- root cause summary
- next remediation step
- whether a rebuild/retry was triggered

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
