---
name: tb_wf_new_project_select_verify
description: Workflow: new project, select, verify Project field, notification email, context log.
---

Policy: `tb_sys_core_constraints`. After any selection change: `tb_sys_context_log`.

Ordered skills:

1. tb_sys_cli_prereqs
2. tb_sys_auth_profile
3. tb_sys_execution_modes
4. tb_sys_core_constraints
5. tb_sys_cloud_selection
6. tb_sys_scope_routing
7. tb_sys_project_application
8. tb_cmd_tau_new_project
9. tb_cmd_tau_select_project — **mandatory** immediately after step 8 with the **same** project name, then `tb_cmd_tau_json_current`; do not skip even if step 8 printed “Selected project”
10. tb_cmd_tau_json_current (verify **Project** matches new name)
11. tb_sys_context_log

**End of turn (new project + apps):** After resource scaffolding, run **`tau clear application`**, **`tau select project --name <name>`**, **`tau json current`** again per `tb_cmd_tau_select_project` so the profile is not left on another project or stuck in an application context.