---
name: tb_cmd_tau_new_service
description: Creates a service resource with path-like protocol routing.
---

## Command card

**Preconditions:** App context if required.

**Command:**

```bash
MSYS_NO_PATHCONV=1 tau --defaults --yes new service --name <svc> --protocol /api
```

**Postconditions:** Git Bash on Windows: prefix with `MSYS_NO_PATHCONV=1` when flags look like paths.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).