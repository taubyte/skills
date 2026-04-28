---
name: tb_cmd_tau_json_current
description: Prints the active profile/cloud/project/application snapshot used for safety checks.
---

## Command card

**Preconditions:** None.

**Command:**

```bash
tau --defaults --yes json current
```

**Postconditions:** Record snapshot in `tb_sys_context_log` after any prior step changed context.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).