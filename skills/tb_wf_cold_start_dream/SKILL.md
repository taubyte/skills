---
name: tb_wf_cold_start_dream
description: Workflow: Node/Docker/tau/dream, auth, execution mode, Dream universe, current JSON.
---

Policy: `tb_sys_core_constraints`. After any selection change: `tb_sys_context_log`.

Ordered skills (Dream branch):

1. tb_sys_cli_prereqs
2. tb_sys_auth_profile
3. tb_sys_execution_modes
4. tb_sys_core_constraints
5. tb_sys_cloud_selection
6. tb_cmd_dream_status_universe
7. tb_cmd_dream_start (if not ready; internally handles `dream start` vs `dream new multiverse --daemon`)
8. tb_cmd_dream_new_universe (only if fresh universe)
9. tb_cmd_tau_select_cloud_universe
10. tb_cmd_tau_json_current
11. tb_sys_context_log