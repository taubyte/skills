---
name: taubyte-execution-modes
description: Defines command mode policy. Non-interactive is default; interactive is opt-in.
---

# Execution Modes

## Default mode

- Non-interactive by default:
  - use `--defaults --yes`
  - provide explicit required flags

## Interactive mode

- Only when user explicitly requests guided prompts.
- Remove `--defaults` and allow CLI prompts.

## Patterns

```bash
# default
tau --defaults --yes <command> <subcommand> <flags...>

# interactive
tau <command> <subcommand>
```
