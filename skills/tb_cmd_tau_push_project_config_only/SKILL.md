---
name: tb_cmd_tau_push_project_config_only
description: Pushes project configuration only with a non-interactive message and build-gate checks.
---

## Command card

**Preconditions:** `notification.email` set; resources defined; context verified with `tau --defaults --yes json current`.

**Command:**

```bash
tau --defaults --yes push project --config-only --message "<message>"
```

**Postconditions:** Poll `tb_cmd_tau_query_builds` until latest Config build completes; then dependent steps. If remote build table is empty, verify via webhook/console fallback before declaring failure.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).