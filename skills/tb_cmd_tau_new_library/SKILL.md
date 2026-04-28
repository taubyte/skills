---
name: tb_cmd_tau_new_library
description: Creates a library resource and repository binding for shared WASM code.
---

## Command card

**Preconditions:** Path `/libraries/<lib>` planned.

**Command:**

```bash
MSYS_NO_PATHCONV=1 tau --defaults --yes new library --name <lib> --path /libraries/<lib> --template empty --generate-repository --no-private --no-embed-token --branch main
```

**Postconditions:** Then `tb_cmd_tau_import_library`. Git Bash on Windows: prefix with `MSYS_NO_PATHCONV=1` when flags look like paths.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).