---
name: tb_cmd_tau_new_application
description: Creates an application when scope routing requires app-scoped resources.
---

## Command card

**Preconditions:** Project selected; multi-app or app-scoped work.

**Command:**

```bash
tau --defaults --yes new application --name <app> --description "<desc>"
```

**Postconditions:** Then `tb_cmd_tau_select_application` if needed.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).