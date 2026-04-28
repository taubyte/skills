---
name: tb_wf_remote_deploy
description: Workflow: select cloud FQDN, push pipeline, query builds/logs.
---

Policy: `tb_sys_core_constraints`. After any selection change: `tb_sys_context_log`.

Ordered skills:

1. `tb_sys_cli_prereqs`
2. `tb_sys_auth_profile`
3. `tb_cmd_tau_select_cloud_fqdn`
4. `tb_cmd_tau_json_current` (must show `Cloud Type: remote`)
5. `tb_sys_project_application` (select/verify project and app scope only if needed)
6. `tb_wf_push_config_then_code`
7. `tb_sys_remote_cloud`
8. `tb_sys_context_log`

## Verification branch

- If `tau query builds` is empty but push succeeded, verify via:
  - GitHub webhook deliveries for pushed repos.
  - Cloud console build/job views.
- Treat webhook + successful push as practical evidence in this environment.