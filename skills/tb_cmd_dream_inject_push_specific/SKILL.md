---
name: tb_cmd_dream_inject_push_specific
description: dream inject push-specific for website/library repos after push-all.
---

## Command card

**Preconditions:** `tb_cmd_dream_inject_push_all` succeeded when both apply. Docker up. Paths/ids match project.

**Command:**

Run `dream inject push-specific --help` for current flags. Example shape:

```bash
dream inject push-specific \
  --universe <universe_name> \
  --repository-id <repo_id> \
  --repository-fullname <owner/repo> \
  <other_required_flags>
```

**Postconditions:** Poll `tb_cmd_tau_query_builds` / `tb_cmd_tau_query_logs`; log in `tb_sys_context_log`. Ordering: `tb_sys_build_runtime_config`.

**Required pair:** `--repository-id` and `--repository-fullname` must be provided together.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).