---
name: tb_wf_dream_inject_and_verify
description: Workflow: Dream build trigger via inject push-specific after GitHub push (no local run).
---

Policy: `tb_sys_core_constraints`. After any selection change: `tb_sys_context_log`.

Ordered skills:

1. `tb_sys_core_constraints` (git HEAD on config/code)
2. `tb_cmd_docker_info`
3. `tb_cmd_dream_inject_push_specific` (websites/libraries when needed)
4. `tb_cmd_tau_query_builds`
5. `tb_cmd_tau_query_logs`
6. `tb_sys_context_log`

See `tb_sys_build_runtime_config` for inject vs remote webhook rules.

## Verification fallback chain

1. Parse inject command output (exit code is not enough).
2. Query builds/logs when rows are available.
3. If builds are delayed, keep the workflow **cloud-only**: re-check push status and keep polling/querying builds/logs. Do **not** curl/run/launch anything locally unless the user explicitly requests local verification.