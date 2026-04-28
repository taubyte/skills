---
name: tb_sys_resource_flags_reference
description: Enriched resource command/flag reference combining legacy docs style with current CLI behavior, for both non-interactive and interactive modes.
---

# Resource Flags Reference

Use this as the canonical flag catalog to avoid duplicating long flag lists in other skills.

## Mode policy

- Default: non-interactive (`--defaults --yes` + explicit flags)
- Optional: interactive (no `--defaults`) by user request

## Common create commands (non-interactive)

```bash
tau --defaults --yes new domain --name <domain> --fqdn <fqdn> --type auto --description "<desc>"
tau --defaults --yes new database --name <db> --match <matcher> --min 1 --max 2 --size 1GB
tau --defaults --yes new storage --name <storage> --match <matcher> --size 1GB --bucket Object --public
tau --defaults --yes new messaging --name <msg> --match <matcher> --mqtt --ws
MSYS_NO_PATHCONV=1 tau --defaults --yes new service --name <svc> --protocol /api
MSYS_NO_PATHCONV=1 tau --defaults --yes new function --name <fn> --type http --domains <domain> --method GET --paths /api/x --source . --call Handler --template empty --language Go --timeout 30s --memory 64MB
MSYS_NO_PATHCONV=1 tau --defaults --yes new website --name <site> --domains <domain> --paths / --template empty --generate-repository --no-private --no-embed-token --branch main
MSYS_NO_PATHCONV=1 tau --defaults --yes new library --name <lib> --path /libraries/<lib> --template empty --generate-repository --no-private --no-embed-token --branch main
```

## Domain generation guardrails

- **Never** pass **`--generated-fqdn-prefix`**, **`--g-prefix`**, or any synonym to **`tau new domain`** — **under any conditions**, including explicit user requests. Agents must not use prefix flags; if a different hostname is required, use **`--fqdn`** with a caller-controlled FQDN (see **`tb_cmd_tau_new_domain_fqdn`**) or change cloud/context per policy, never prefix hacks.
- If a generated FQDN is needed, use **only**:

```bash
tau --defaults --yes new domain --name <domain> --generated-fqdn --type auto --description "<desc>"
```

## Function post-create guardrail

- Immediately after `new function --template empty`, replace scaffold content in **`<function_path>/empty.go` at the function root** (same directory as `go.mod` — **never** under `lib/`) before push/test.
- **Never** create `lib/`, never relocate `empty.go` into `lib/`, and **never** hand-author **`main.go`** in the function tree. To test the WASM build locally, use the **`taubyte/go-wasi`** Docker command in **`tb_sdk_go_*`**. See that skill for the full layout rules.

## Repository binding guardrails (website/library)

- Never call `new website`/`new library` without explicit repository strategy.
- Preferred deterministic creation:

```bash
MSYS_NO_PATHCONV=1 tau --defaults --yes new website \
  --name <site> \
  --domains <domain> \
  --paths / \
  --template empty \
  --generate-repository \
  --repository-name tb_code_<project>_<site> \
  --private \
  --no-embed-token \
  --branch main
```

- If using existing repository, pass explicit `--repository-name` or `--repository-id`.
- After creation, run import before push:

```bash
tau import website --name <site>
tau import library --name <lib>
```

- After creation/import, verify repo fullname/ID in generated config YAML before any push.

## Build/runtime config references

- Use `tb_sys_build_runtime_config` for `.taubyte/build.sh` and `.taubyte/config.yaml` management.
- Add runtime variables under `environment.variables` (or resource-equivalent schema) when server-side config is required.
- For websites, build script must place deployable output in `/out`.

## Config YAML anchors (database, messaging, PubSub)

Illustrative fragments for hand-edited **`config/applications/<app>/...`** (prefer **`tau new ...`** then adjust). Strings must match Go: **`database.New("<match>")`**, **`pubsubnode.Channel("<channel>")`**, and PubSub **`trigger.channel`**.

**Database** (`databases/<name>.yaml`):

```yaml
match: appdata
useRegex: false
access:
  network: all
storage:
  size: 1GB
replicas:
  min: 1
  max: 2
```

**Messaging** (`messaging/<name>.yaml`):

```yaml
local: false
channel:
  regex: false
  match: chat
bridges:
  mqtt:
    enable: false
  websocket:
    enable: true
```

**PubSub consumer function** (snippet):

```yaml
trigger:
  type: pubsub
  local: false
  channel: chat
execution:
  call: handleChatEvents
```

## Legacy-style interactive note

- In interactive mode, prompts are expected for visibility, template choice, embed-token, path/domain/method selections, and confirmation.
