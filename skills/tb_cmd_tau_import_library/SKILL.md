---
name: tb_cmd_tau_import_library
description: Imports an existing library resource repository into the selected Taubyte project.
---

## Command card

**Preconditions:** `tau new library` done; **Project** verified.

**Command:**

```bash
tau --defaults --yes import library --name <lib>
```

**Postconditions:** Verify `github.fullname` in config YAML and that project context is still correct (`tau --defaults --yes json current`).

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).