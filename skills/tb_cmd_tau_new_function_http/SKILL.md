---
name: tb_cmd_tau_new_function_http
description: Creates an HTTP function scaffold with explicit route/method/runtime settings.
---

## Command card

**Preconditions:** Domain exists; paths/methods known.

**Command:**

```bash
MSYS_NO_PATHCONV=1 tau --defaults --yes new function --name <fn> --type http --domains <domain> --method GET --paths /api/x --source . --call Handler --template empty --language Go --timeout 30s --memory 64MB
```

**Postconditions:** Edit root `empty.go` only; ensure `execution.call` matches `//export Handler`; see `tb_sdk_go_function_http`. Git Bash on Windows: prefix with `MSYS_NO_PATHCONV=1` when flags look like paths.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).