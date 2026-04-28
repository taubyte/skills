---
name: tb_cmd_tau_new_storage
description: Creates an app-scoped storage resource with bucket policy and matcher.
---

## Command card

**Preconditions:** App context if required.

**Command:**

```bash
tau --defaults --yes new storage --name <storage> --match <matcher> --size 1GB --bucket Object --public
```

**Postconditions:** Update context log.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).