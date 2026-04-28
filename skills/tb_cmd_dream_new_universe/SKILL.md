---
name: tb_cmd_dream_new_universe
description: dream new universe (resets Dream-side state).
---

## Command card

**Preconditions:** User accepts fresh universe.

**Command:**

```bash
dream new universe <name>
```

**Postconditions:** Then `tb_cmd_tau_select_cloud_universe` and recreate resources per `tb_sys_core_constraints`.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).