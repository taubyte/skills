---
name: tb_cmd_dream_status_universe
description: dream status universe.
---

## Command card

**Preconditions:** Dream process reachable.

**Command:**

```bash
dream status universe <name>
```

**Postconditions:** Use `default` unless user requested another.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).