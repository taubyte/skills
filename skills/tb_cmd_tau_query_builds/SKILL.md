---
name: tb_cmd_tau_query_builds
description: Queries recent build rows and job ids for post-push verification.
---

## Command card

**Preconditions:** After push/inject.

**Command:**

```bash
tau query builds --since 1h
```

**Postconditions:** Capture job IDs in `tb_sys_context_log`.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).