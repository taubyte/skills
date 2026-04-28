---
name: tb_cmd_tau_select_application
description: Selects the active application context for app-scoped resource operations.
---

## Command card

**Preconditions:** Application exists; scope requires app context.

**Command:**

```bash
tau --defaults --yes select application --name <app>
tau --defaults --yes json current
```

**Postconditions:** Log active application.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).