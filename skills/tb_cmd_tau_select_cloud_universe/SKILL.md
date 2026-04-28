---
name: tb_cmd_tau_select_cloud_universe
description: Selects a Dream universe cloud target and verifies local context.
---

## Command card

**Preconditions:** Dream universe reachable (`tb_cmd_dream_status_universe`).

**Command:**

```bash
tau --defaults --yes select cloud --universe <universe_name>
tau --defaults --yes json current
```

**Postconditions:** Log universe name in context log.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).