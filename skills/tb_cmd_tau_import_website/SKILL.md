---
name: tb_cmd_tau_import_website
description: Imports an existing website resource repository into the selected Taubyte project.
---

## Command card

**Preconditions:** `tau new website` done; **Project** verified.

**Command:**

```bash
tau --defaults --yes import website --name <site>
```

**Postconditions:** Verify `github.fullname` in config YAML and that project context is still correct (`tau --defaults --yes json current`).

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).