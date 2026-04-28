---
name: tb_wf_dream_inject_and_verify
description: Workflow: docker info, push-all before push-specific, inject from project root.
---

Policy: `tb_sys_core_constraints`. After any selection change: `tb_sys_context_log`.

Ordered skills:

1. `tb_sys_core_constraints` (git HEAD on config/code)
2. `tb_cmd_docker_info`
3. `tb_cmd_dream_inject_push_all`
4. `tb_cmd_dream_inject_push_specific` (websites/libraries when needed)
5. `tb_cmd_tau_query_builds`
6. `tb_cmd_tau_query_logs`
7. `tb_cmd_tau_query_domain`
8. `tb_sys_context_log`

See `tb_sys_build_runtime_config` for inject vs remote webhook rules.

## Verification fallback chain

1. Parse inject command output (exit code is not enough).
2. Query builds/logs when rows are available.
3. If builds are delayed, confirm domain/resources and run local URL checks (`tb_wf_local_url_verify`) where applicable.