---
name: tb_cmd_dream_inject_push_all
description: dream inject push-all (explicit-only; do not use unless user asks).
---

## Command card

**Preconditions:** config/ and code/ repos have git HEAD; Docker up; path = project root; explicit project id available from `config/config.yaml`.

**Command:**

```bash
dream inject push-all --path <absolute_project_root> --project-id <project_id> --universe <universe_name>
```

**Postconditions:** Poll builds/logs.

**Policy:** Do **not** run `push-all` by default. The default local Dream build trigger is `dream inject push-specific` after GitHub push. Use `push-all` only when the user explicitly asks for a full sync or when a documented Dream workflow requires it for their specific setup.

**Validation (required):**
- Do not trust exit code alone (can be `0` on failure).
- Parse stdout/stderr for failure markers (`failed`, `inject`, missing path/resource errors).
- If failure text appears, treat the step as failed and recover before continuing.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).