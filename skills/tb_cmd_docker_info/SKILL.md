---
name: tb_cmd_docker_info
description: docker info: verify engine for Dream/inject.
---

## Command card

**Preconditions:** Docker CLI installed.

**Command:**

```bash
docker info
```

**Postconditions:** If fails, stop Dream inject path per `tb_sys_core_constraints`.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).