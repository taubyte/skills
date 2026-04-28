---
name: tb_cmd_dream_status_substrate
description: dream status substrate (HTTP port discovery).
---

## Command card

**Preconditions:** Dream local verify.

**Command:**

```bash
dream status substrate
```

**Postconditions:** Use port with curl / browser (see `tb_sys_local_host_launch`).

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).