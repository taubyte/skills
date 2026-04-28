---
name: tb_cmd_tau_new_database
description: Creates an app-scoped database resource with matcher/replica sizing.
---

## Command card

**Preconditions:** App selected if resource is app-scoped.

**Command:**

```bash
tau --defaults --yes new database --name <db> --match <matcher> --min 1 --max 2 --size 1GB
```

**Postconditions:** Align `match` with `database.New` in code.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).