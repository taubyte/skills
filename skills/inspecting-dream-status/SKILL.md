---
name: inspecting-dream-status
description: Inspects a running Dream local cloud — verifies a universe is up, discovers gateway/substrate ports and URLs, and explains the wrong-universe failure mode. Use when probing Dream readiness, finding the gateway port for HTTP function tests, debugging `universe does not exist`, or confirming Dream is reachable before running `tau` or `dream inject`.
---

# Inspecting Dream Status

## When to use

- "Is Dream up?"
- "What port is the Dream gateway on?"
- "Find the substrate URL"
- `tau` commands fail with cloud-related errors despite Dream being started
- `dream` reports `universe \`xxx\` does not exist`

## Core commands

All `dream status` subcommands take the universe as a **positional argument**. There is also a `--universe` / `-u` / `--to` flag (defaults to `blackhole`), but for clarity prefer the positional form.

```bash
dream status universe   <universe>   # overall universe state + service ports
dream status gateway    <universe>   # HTTP gateway endpoint (used to hit functions)
dream status substrate  <universe>   # substrate (build/runtime) endpoint
```

Replace `<universe>` with whatever the multiverse was started with — typically:

- `default` if started via `dream start`
- `blackhole` if started via `dream new multiverse` without `--universes`

## Quick readiness check

```bash
dream status universe default
```

A successful call lists the running services and their assigned ports. If it errors with `universe \`<name>\` does not exist`, the universe name is wrong — see the table below.

## Finding the gateway port (for curl tests)

```bash
# `dream status gateway` often prints a blank line before `@ http://...`; awk '/@ http/{print $3}' can be empty.
GW=$(dream status gateway default | grep -Eo 'http://[0-9.]+:[0-9]+' | head -n1)
echo "$GW"
# example: http://127.0.0.1:<gateway_port>
```

Combine with a domain FQDN to hit an HTTP function:

```bash
FQDN=$(awk '/^fqdn:/{print $2; exit}' config/domains/<domain>.yaml)
curl -i -H "Host: $FQDN" "$GW/<path>"
```

## Universe-name failure mode

`dream status` always prints results for **exactly** the universe name passed in. Common mismatches:

| Started Dream with | Use universe name | Wrong name fails with |
| --- | --- | --- |
| `dream start` (no flags) | `default` | `universe \`blackhole\` does not exist` |
| `dream new multiverse` (no flags) | `blackhole` | `universe \`default\` does not exist` |
| `dream start --universes dev` | `dev` | error if you query `default` |

Recovery: re-check how Dream was started, then call `dream status universe <correct_name>`. See [starting-dream-locally](../starting-dream-locally/SKILL.md).

## Status output (what to look for)

`dream status universe <u>` typically shows:

- A list of services (auth, seer, tns, patrick, monkey, substrate, gateway, ...) each with assigned host:port.
- Lifetime / uptime info per service.

`dream status gateway <u>` shows the inbound HTTP endpoint clients use; pair with the `Host:` header to route to a specific FQDN.

`dream status substrate <u>` shows the substrate HTTP endpoint; useful when you need to talk to substrate APIs directly (rare in normal flows).

## Gotchas

- **Gateway URL parsing:** `dream status gateway <u>` often prints a **blank line** before `@ http://...`. Parsers like `awk '/@ http/{print $3}'` can return **empty**; prefer `grep -Eo 'http://[0-9.]+:[0-9]+' | head -n1` (same pattern as [verifying-taubyte-functions](../verifying-taubyte-functions/SKILL.md)).
- The default value of the `--universe` / `-u` flag is `blackhole`, even on builds where `dream start` defaults to `default`. **Pass the universe explicitly** to avoid silent mismatches.
- A status call that hangs usually means the multiverse process exited; check the terminal running `dream start` / `dream new multiverse --daemon`.
- Ports are dynamic per run. **Don't hard-code ports across sessions** — re-discover via `dream status` each time.

## Related skills

- `starting-dream-locally` — bring up Dream and pick a universe name
- `triggering-dream-builds` — uses the same universe argument
- `verifying-taubyte-functions` — uses the gateway port + Host header to hit a function
- `diagnosing-dream-builds` — uses the discovered ports for direct jobs API calls
