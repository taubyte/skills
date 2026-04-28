---
name: tb_cmd_dream_start
description: dream start (daemon; do not wait for exit).
---

## Command card

**Preconditions:** Docker daemon up.

**Command (version-aware):**

First detect which Dream CLI you have (stable vs dev):

```bash
dream --help
```

- If `dream --help` shows a `start` command (dev/newer Dream), run:

```bash
dream start
```

- If `dream --help` does **not** show `start` (stable/older Dream), start Dream via multiverse:

```bash
dream new multiverse --daemon --universes <universe_name>
```

Use `default` as `<universe_name>` unless the user requested another.

**Postconditions:** Continue with `tb_cmd_dream_status_universe` readiness polls.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).