---
name: tb_cmd_dream_status_gateway
description: dream status gateway.
---

## Command card

**Preconditions:** Debugging routing.

**Command:**

```bash
dream status gateway <name>
```

**Postconditions:** Log output in context log.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).