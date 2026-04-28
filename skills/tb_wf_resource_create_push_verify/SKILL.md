---
name: tb_wf_resource_create_push_verify
description: Workflow: scope, project, resource creation, flags ref, push config/code, query builds.
---

Policy: `tb_sys_core_constraints`. After any selection change: `tb_sys_context_log`.

Ordered skills:

1. `tb_sys_cli_prereqs`
2. `tb_sys_auth_profile`
3. `tb_sys_cloud_selection`
4. `tb_sys_scope_routing`
5. `tb_sys_project_application`
6. `tb_cmd_tau_json_current`
7. `tb_sys_resource_creation` (use `tb_sys_resource_flags_reference` + `tb_cmd_tau_new_*` as needed)
8. `tb_sys_build_runtime_config`
9. `tb_wf_push_config_then_code`
10. `tb_sys_debug_recovery` (on failure)

## Scope checks

- Project-scoped resources (for example global domain) may not resolve while app scope is selected.
- If resource not found appears unexpectedly, use `tb_sys_project_application` scope correction guard before retry.