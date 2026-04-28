---
name: tb_cmd_tau_query_job
description: Queries a specific build job status and metadata by job id.
---

## Command card

**Preconditions:** Have job id.

**Command:**

```bash
tau query job --jid <job_id>
```

**Postconditions:** Record status in context log.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).