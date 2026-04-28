---
name: tb_wf_local_url_verify
description: Workflow: hosts mapping, dream inject if needed, substrate port, curl URLs.
---

Policy: `tb_sys_core_constraints`. After any selection change: `tb_sys_context_log`.

Ordered skills:

1. tb_sys_cloud_selection
2. tb_wf_dream_inject_and_verify (if code not yet injected)
3. tb_sys_hosts_file
4. tb_sys_local_host_launch
5. tb_sys_context_log