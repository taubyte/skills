---
name: tb_cmd_tau_push_website
description: Triggers website repo push/build with scope-aware prechecks.
---

## Command card

**Preconditions:** Config build success; website exists on target cloud (import/config push already applied); scope verified (project vs application).

**Command:**

```bash
tau --defaults --yes push website --name <website> --message "<message>"
```

**Postconditions:** Poll builds; git fallback only per `tb_sys_core_constraints`.

**Scope note:** If `website <name> not found`, re-check `tau --defaults --yes json current`, then use `tau --defaults --yes clear application` for project-scoped website pushes or select the correct app for app-scoped website resources.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).