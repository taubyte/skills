---
name: tb_cmd_tau_query_logs
description: Retrieves logs for a specific job id to diagnose build/runtime failures.
---

## Command card

**Preconditions:** Have job id.

**Command:**

```bash
tau query logs --jid <job_id>
```

**Postconditions:** Summarize root cause in context log.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).