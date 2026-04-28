---
name: starting-dream-locally
description: Starts a Dream local Taubyte cloud (multiverse) using either the newer `dream start` workflow or the older `dream new multiverse` workflow, and explains the default universe naming (`default` for `dream start`, `blackhole` for `dream new multiverse`). Use when bringing up, restarting, or daemonizing a local Dream cloud, or when `dream --help` shows a different shape than expected.
---

# Starting Dream Locally

## When to use

- "Start Dream"
- "Bring up the local cloud"
- "Daemonize Dream"
- `dream` commands fail because no universe is running
- `dream --help` looks different from what was expected

## Preflight

```bash
docker version
dream --help        # determines which workflow this binary supports
```

If the help output shows a `start` subcommand, use the **newer workflow**. If it shows `new multiverse`, use the **older workflow**.

## Newer workflow — `dream start`

Run in its own terminal so logs stream visibly:

```bash
dream start
```

Common options (from `dream start --help`):

| Flag | Purpose |
| --- | --- |
| `--universes` / `-u` | Comma-separated list of universes (default: a single universe named `default`) |
| `--branch` / `-b` | Default git branch (default: `main`) |
| `--debug` | Extra output for function calls |
| `--public` | Expose APIs on all interfaces (instead of `127.0.0.1`) |

**Default universe name with `dream start`: `default`.** Verify with:

```bash
dream status universe default
```

Custom universe example:

```bash
dream start --universes dev
dream status universe dev
```

To create more universes after Dream is running:

```bash
dream new universe <name>
```

## Older workflow — `dream new multiverse`

```bash
dream new multiverse --daemon
dream new multiverse --daemon --universes blackhole
dream new multiverse --daemon --universes blackhole,dev
```

Options:

| Flag | Purpose |
| --- | --- |
| `--daemon` | Run in background |
| `--keep` | Persist universe state under `$HOME/.cache/dreamland` instead of `/tmp` |
| `--listen-on-all` / `-L` | Bind Dream HTTP clients on `0.0.0.0` instead of `127.0.0.1` |
| `--branch` / `-b` | Default git branch (default: `main`) |
| `--universes` | Comma-separated list of universes |

**Default universe name with `dream new multiverse`: `blackhole`** (unless `--universes` overrides it). Verify with:

```bash
dream status universe blackhole
```

## Universe naming rule (critical)

`dream status` and most `dream` subcommands take the **universe** as a positional argument. The name **must match exactly** what the multiverse was started with:

| Started with | Default universe name | Status check |
| --- | --- | --- |
| `dream start` (no flags) | `default` | `dream status universe default` |
| `dream new multiverse` (no flags) | `blackhole` | `dream status universe blackhole` |
| `dream start --universes dev` | `dev` | `dream status universe dev` |
| `dream new multiverse --universes dev` | `dev` | `dream status universe dev` |

Querying the wrong universe fails with `universe \`blackhole\` does not exist` (or similar).

## Recommended local default

Unless the user requests otherwise, use the **newer workflow with the `default` universe**:

```bash
dream start
# in another terminal:
dream status universe default
```

This keeps every downstream skill (`tau select cloud --universe default`, `dream inject ... default`, etc.) working without ambiguity.

## Gotchas

- **`dream start` is foreground by design.** It streams logs in that terminal. Run it in a dedicated terminal/tab; don't background it with `&` unless you have log redirection set up.
- **Dream binary version drift.** If you've been working off two different machines, the available subcommands may differ. Always run `dream --help` first when on a fresh machine.
- **Custom universes don't auto-discover.** If you start with `--universes foo,bar`, you must use `foo` or `bar` (not `default`/`blackhole`) for every later `dream status` / `dream inject` / `tau select cloud --universe` call.

## Related skills

- `inspecting-dream-status` — verify ports/URLs once Dream is up
- `triggering-dream-builds` — push GitHub state into the local cloud
- `selecting-taubyte-context` — point `tau` at the running universe
