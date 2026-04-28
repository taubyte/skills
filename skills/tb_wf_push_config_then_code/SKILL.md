---
name: tb_wf_push_config_then_code
description: Workflow: config-only push, wait builds, code push, optional website/library push.
---

Policy: `tb_sys_core_constraints`. After any selection change: `tb_sys_context_log`.

Ordered skills:

1. `tb_cmd_tau_json_current`
2. `tb_sys_push_build_verify` (pre-push checks)
3. `tb_cmd_tau_push_project_config_only` → `tb_cmd_tau_query_builds` → `tb_cmd_tau_query_logs` (wait for config build success)
4. `tb_cmd_tau_push_project_code_only` → poll builds/logs again
5. `tb_cmd_tau_push_website` / `tb_cmd_tau_push_library` as needed
6. `tb_sys_context_log`

**Alternative (single step):** only if the user explicitly wants one push: `tb_cmd_tau_push_project_full` — still follow build polling; default remains split config/code per `tb_sys_core_constraints`.

## Remote fallback

- If remote `tau query builds` stays empty after confirmed pushes, verify via webhook deliveries and cloud console before rollback.
- Website/library push errors (`not found`) usually mean config not applied yet or scope mismatch; re-run config push and verify app/project scope before retrying.