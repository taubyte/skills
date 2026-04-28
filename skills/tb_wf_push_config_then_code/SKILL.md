---
name: tb_wf_push_config_then_code
description: Workflow: push config+code to GitHub; remote waits for cloud build; Dream triggers inject push-specific (no local run).
---

Policy: `tb_sys_core_constraints`. After any selection change: `tb_sys_context_log`.

Ordered skills:

1. `tb_cmd_tau_json_current`
2. `tb_sys_push_build_verify` (pre-push checks)
3. `tb_sys_build_runtime_config` (if `.taubyte/*` changed)
4. Push all changed repos to GitHub (config/code/website/library as applicable)
5. Remote: wait for webhook builds; Dream: `tb_wf_dream_inject_and_verify` (inject push-specific only)
6. `tb_cmd_tau_query_builds` → `tb_cmd_tau_query_logs`
7. `tb_sys_context_log`

**Alternative:** only if the user explicitly insists on `tau push ...` flows, use `tb_cmd_tau_push_project_full` / `tb_cmd_tau_push_project_config_only` / `tb_cmd_tau_push_project_code_only`, but the default remains **GitHub push → cloud build**.

## Remote fallback

- If remote `tau query builds` stays empty after confirmed pushes, verify via webhook deliveries and cloud console before rollback.
- Website/library push errors (`not found`) usually mean config not applied yet or scope mismatch; re-run config push and verify app/project scope before retrying.