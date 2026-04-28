---
name: tb_cmd_tau_new_website
description: Creates a website resource with explicit domain/path and repository strategy.
---

## Command card

**Preconditions:** Domain attached; repo strategy explicit.

**Command:**

```bash
MSYS_NO_PATHCONV=1 tau --defaults --yes new website --name <site> --domains <domain> --paths / --template empty --generate-repository --no-private --no-embed-token --branch main
```

**Postconditions:** Then `tb_cmd_tau_import_website`. Git Bash on Windows: prefix with `MSYS_NO_PATHCONV=1` when flags look like paths.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).