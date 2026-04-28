---
name: tb_cmd_tau_push_project_code_only
description: Pushes project code only after config readiness is verified.
---

## Command card

**Preconditions:** Latest config build succeeded; context verified with `tau --defaults --yes json current`.

**Command:**

```bash
tau --defaults --yes push project --code-only --message "<message>"
```

**Postconditions:** Poll builds/logs; update context log.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).