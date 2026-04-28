---
name: authoring-taubyte-function-types
description: Authoritative reference for the four Taubyte function trigger types — `http`, `https`, `p2p`, and `pubsub` — covering the per-type `tau new function` CLI flag matrix from `tau new function --help`, the resulting YAML shape under `trigger:`, the matching Go SDK handler entry points, and the "must-match" wiring strings (channel, protocol, command, call). Use when creating a function whose trigger isn't plain HTTPS, when picking between `http` and `https`, when wiring a PubSub-triggered function to a messaging channel, or when reviewing a function YAML to confirm flag/YAML/Go consistency.
---

# Authoring Taubyte Function Types

## When to use

- "Create a PubSub-triggered function" / "P2P service / handler"
- Choosing between `--type http` and `--type https`
- Reviewing a function YAML's `trigger:` section
- Wiring a function to a messaging channel or P2P protocol
- Confirming the four "must-match" strings (`call`, `channel`, `protocol`, `command`) line up across CLI flag → YAML → Go

## The four trigger types (CLI-authoritative)

From `tau new function --help`:

```text
--type    one of: [http, https, p2p, pubsub]
```

| Type | Use for |
| --- | --- |
| `https` | Default for HTTP APIs and webhooks served over TLS (also what tau.how uses in canonical examples) |
| `http`  | Same as https but plain HTTP — typically only on Dream local where TLS isn't needed |
| `p2p`   | Peer-to-peer service handler over libp2p (`protocol` + `command`) |
| `pubsub`| Message consumer subscribed to a messaging channel |

Generic flags that apply to **all** trigger types:

| Flag | Purpose |
| --- | --- |
| `--name` / `-n` | Resource name (also positional) |
| `--description` / `-d` | Free-text description |
| `--language` / `--lang` | One of `[Rust, Go, Assembly_Script]` |
| `--source` | `.` for inline (code repo) or `libraries/<library>` |
| `--call` / `--ca` | The exported function symbol — must match `//export <name>` in source |
| `--memory` / `--me` + `--memory-unit` | e.g. `--memory 64MB` or `--memory 64 --memory-unit MB` |
| `--timeout` / `--ttl` | e.g. `30s` |
| `--tags` / `-t` | Repeatable |
| `--template` + `--use-template` | Pick a starter template |
| `--local` | Boolean — restrict to local-only execution |

## `--type https` (and `--type http`)

### Type-specific CLI flags

```text
--domains          (list, repeatable)
--generate-domain, -g
--method, -m       one of: [GET, POST, PUT, DELETE, CONNECT, HEAD, OPTIONS, TRACE, PATCH]
--paths, -p        (list, repeatable)
```

`--generate-domain` creates a fresh `*.<universe>.localtau` (Dream) or generated remote domain instead of attaching to existing `--domains`.

### Canonical create command

```bash
MSYS_NO_PATHCONV=1 tau --defaults --yes new function <fn> \
  --type https \
  --method GET \
  --paths /api/v1/items \
  --domains <domain_name> \
  --language Go \
  --source . \
  --memory 64MB \
  --timeout 30s \
  --call ListItems \
  --template empty
```

### Resulting YAML

```yaml
description: <desc>
source: .
trigger:
  type: https
  method: GET
  paths:
    - /api/v1/items
domains:
  - <domain_name>
execution:
  timeout: 30s
  memory: 64MB
  call: ListItems
```

### Go entry point

```go
//export ListItems
func ListItems(e event.Event) uint32 {
    h, err := e.HTTP()
    if err != nil { return 1 }
    // headers → write body → Return(status), in that order
    h.Headers().Set("Content-Type", "application/json")
    h.Write([]byte(`[]`))
    h.Return(200)
    return 0
}
```

See [writing-taubyte-functions](../writing-taubyte-functions/SKILL.md) for the full HTTP event API.

### Rules

- **One function per `(path, method)` combination** — never comma-separated methods. Create separate function resources for each method on the same path.
- Multiple `--paths` on the **same** function-and-method are allowed (the function handles all listed paths).

## `--type pubsub`

### Type-specific CLI flags

```text
--channel, --ch    (the channel string to subscribe to)
```

### Canonical create command

```bash
tau --defaults --yes new function <fn> \
  --type pubsub \
  --channel chat \
  --language Go \
  --source . \
  --memory 64MB \
  --timeout 30s \
  --call OnChat \
  --template empty
```

### Resulting YAML

```yaml
description: <desc>
source: .
trigger:
  type: pubsub
  channel: chat
execution:
  timeout: 30s
  memory: 64MB
  call: OnChat
```

### Go entry point

```go
//export OnChat
func OnChat(e event.Event) uint32 {
    ps, err := e.PubSub()
    if err != nil { return 1 }
    data, _ := ps.Data()
    ch, _   := ps.Channel()
    _ = ch.Name()
    _ = data
    return 0
}
```

### Wiring (three strings must match)

For the consumer to actually fire, **the same channel string** must appear in three places:

| Where | Field |
| --- | --- |
| Messaging YAML | `channel.match: chat` |
| Function YAML | `trigger.channel: chat` |
| Producer Go code | `pubsubnode.Channel("chat").Publish(...)` |

Create the **messaging resource first** (or in lockstep) so the channel exists before any consumer subscribes:

```bash
tau --defaults --yes select application --name <app>
tau --defaults --yes new messaging chat --match chat --mqtt --ws
```

PubSub-triggered functions usually **don't need a domain** (no HTTP route). Skip `--domains` / `--paths` / `--method`.

## `--type p2p`

### Type-specific CLI flags

```text
--protocol, --pr   (libp2p protocol path, e.g. "/myproto/v1")
--command, --cmd   (the command name handled by this function)
```

### Canonical create command

```bash
MSYS_NO_PATHCONV=1 tau --defaults --yes new function <fn> \
  --type p2p \
  --protocol /myproto/v1 \
  --command ping \
  --language Go \
  --source . \
  --memory 64MB \
  --timeout 5s \
  --call OnP2P \
  --template empty
```

### Resulting YAML

```yaml
description: <desc>
source: .
trigger:
  type: p2p
  protocol: /myproto/v1
  command: ping
execution:
  timeout: 5s
  memory: 64MB
  call: OnP2P
```

### Go entry point

```go
//export OnP2P
func OnP2P(e event.Event) uint32 {
    p, err := e.P2P()
    if err != nil { return 1 }
    cmd, _   := p.Command()
    proto, _ := p.Protocol()
    data, _  := p.Data()
    p.Write([]byte("ack"))
    _ = cmd; _ = proto; _ = data
    return 0
}
```

### Wiring (caller side)

Other functions (often HTTP) call this P2P endpoint via `p2p/node`:

```go
import p2pnode "github.com/taubyte/go-sdk/p2p/node"

svc := p2pnode.New("/myproto/v1")
cmd, err := svc.Command("ping")
resp, err := cmd.Send([]byte("payload"))
```

The protocol + command strings must match the YAML exactly.

## Side-by-side flag matrix

| Flag | https/http | p2p | pubsub |
| --- | --- | --- | --- |
| `--method` | required | — | — |
| `--paths` | required (≥1) | — | — |
| `--domains` / `--generate-domain` | required (one of) | — | — |
| `--protocol` | — | required | — |
| `--command` | — | required | — |
| `--channel` | — | — | required |
| `--call` | required | required | required |
| `--source` | required | required | required |
| `--memory`, `--timeout` | required | required | required |

## Choosing between `http` and `https`

- **Production / remote clouds**: use `--type https`. tau.how docs use `https` as the canonical.
- **Dream local**: either works. Remote cloud generated domains are TLS-issued automatically; on Dream local you may not have certs, so plain `http` test paths via the gateway are the norm — but the **trigger type itself** is fine either way; what matters at test time is the URL scheme used to curl, not the YAML.
- **Mixing**: don't have two functions with `https` and `http` for the **same** `(domain, method, path)` combination — one of them will be unreachable.

## Workflow checklist

```
Function-type authoring progress:
- [ ] Decide trigger type (https | http | p2p | pubsub)
- [ ] For pubsub: messaging resource exists with channel.match = <channel>
- [ ] Run tau new function with the type-specific flags above
- [ ] Confirm the YAML under config/.../<fn>.yaml has the expected trigger.<fields>
- [ ] Edit the function root empty.go (next to go.mod) — never under lib/
- [ ] //export <name> matches execution.call exactly
- [ ] For pubsub/p2p: wiring strings match across YAML and Go (channel / protocol / command)
- [ ] tau push project --config-only -m "..."  (then wait for config build)
- [ ] tau push project --code-only -m "..."   (or push library if source: libraries/<lib>)
- [ ] Dream: dream inject push-specific for the relevant repo(s)
- [ ] Verify with curl (https/http), with a producer call (pubsub), or via p2p caller (p2p)
```

## Gotchas

- **Forgetting `--method` on https/http**: command fails or scaffolds with no method, producing a non-routable function. Pass it explicitly.
- **Channel mismatch silently breaks pubsub**: triple-check the literal across messaging YAML, function YAML, and Go producer/consumer.
- **PubSub function with `--domains`**: harmless but misleading — the function doesn't serve HTTP. Omit domain flags for pure consumers.
- **P2P protocol typos**: `/myproto/v1` and `/myproto/V1` are different protocols; libp2p won't fall back. Prefer lowercase, versioned paths.
- **Multiple methods via comma in `--method`** isn't supported — split into separate function resources per method.
- **`--type http` on a remote cloud**: usually rejected at deploy because the cloud's gateway expects TLS. Use `https` for remote.
- **`--call` not exported in code**: function compiles but the cloud can't dispatch — produces "function not found"-style runtime errors. Match `//export <name>` to `execution.call` exactly.

## Related skills

- `creating-taubyte-resources` — broader `tau new` reference (domains, websites, dbs, etc.)
- `writing-taubyte-functions` — full Go SDK handler patterns for every trigger type
- `editing-taubyte-resources` — change paths/methods/channel after creation
- `configuring-taubyte-build-runtime` — `.taubyte/build.sh` + `.taubyte/config.yaml` for any function type
- `pushing-taubyte-projects` — push order: config first, code/library second
- `triggering-dream-builds` — bring a new function into Dream
