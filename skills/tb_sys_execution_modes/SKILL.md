---
name: tb_sys_execution_modes
description: Defines command mode policy. Non-interactive is default; interactive is opt-in.
---

# Execution Modes

## Default mode

- Non-interactive by default:
  - use `--defaults --yes`
  - provide explicit required flags
  - for all `tau push ...` commands, include `--message "<message>"`

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

## Push safety pattern

```bash
tau --defaults --yes push project --config-only --message "<message>"
tau --defaults --yes push project --code-only --message "<message>"
tau --defaults --yes push website --name <website> --message "<message>"
tau --defaults --yes push library --name <library> --message "<message>"
```

- Do not rely on prompt-based commit messages in automation.
- If context-sensitive operation follows `tau clear application`, re-run:
  ```bash
  tau --defaults --yes json current
  ```
