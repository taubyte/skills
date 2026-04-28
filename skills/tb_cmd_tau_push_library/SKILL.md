---
name: tb_cmd_tau_push_library
description: Triggers library repo push/build with scope and config prerequisites.
---

## Command card

**Preconditions:** Config build success; library exists on target cloud (import/config push already applied); scope verified (project vs application).

**Command:**

```bash
tau --defaults --yes push library --name <library> --message "<message>"
```

**Postconditions:** Poll builds; git fallback only per `tb_sys_core_constraints`.

**Scope note:** If `library <name> not found`, verify context (`tau --defaults --yes json current`) and ensure the config build already created that library on the selected cloud.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).