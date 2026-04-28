---
name: tb_cmd_tau_push_project_full
description: Performs one-step config+code push when split sequencing is intentionally skipped.
---

## Command card

**Preconditions:** User explicitly wants single `tau push project`; understand you skip separate config/code gate ordering.

**Command:**

```bash
tau --defaults --yes push project --message "<message>"
```

**Postconditions:** Poll builds/logs; update context log; still respect website/library ordering per `tb_sys_core_constraints`.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).