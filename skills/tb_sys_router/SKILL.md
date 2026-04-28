---
name: tb_sys_router
description: Entry Taubyte skill: intent to tb_wf_* load order; Dream default; no duplicated policy.
---

# Router

Load **one** primary `tb_wf_*` for the user goal, then follow its ordered skill list. Policy lives in `tb_sys_core_constraints` only.

## Mandatory gate order (always)

1. `tb_sys_cli_prereqs`
2. `tb_sys_auth_profile` (when GitHub / push / clone / generated repos)
3. `tb_sys_execution_modes`
4. `tb_sys_core_constraints`
5. `tb_sys_cloud_selection`
6. `tb_sys_scope_routing`
7. `tb_sys_project_application`
8. `tb_sys_context_log` (initialize / update after any context change)
9. `tb_sys_resource_creation` (when mutating resources)
10. Relevant `tb_sdk_go_*` when editing Go/WASM
11. `tb_sys_build_runtime_config` (when `.taubyte/*` or Dream inject applies)
12. `tb_sys_push_build_verify` then Dream **or** remote path:
    - Dream: `tb_wf_dream_inject_and_verify`
    - Remote: `tb_sys_remote_cloud`
13. `tb_sys_debug_recovery` on failure

## Intent → workflow (pick one)

| User intent | Start here |
|-------------|--------------|
| First-time / env check | `tb_wf_cold_start_dream` |
| New Taubyte project | `tb_wf_new_project_select_verify` |
| Create resource + deploy | `tb_wf_resource_create_push_verify` |
| Dream inject only | `tb_wf_dream_inject_and_verify` |
| Push config then code | `tb_wf_push_config_then_code` (optional `tb_cmd_tau_push_project_full` when user insists on one push) |
| Go WASM / SDK coding only | `tb_wf_go_wasm_authoring` |
| Remote FQDN cloud | `tb_wf_remote_deploy` |
| Self-host cloud (Spore) | `tb_sys_spore_drive` |

## New project hard gate (routing only)

Before **import**, **push**, or **`tau new`** resources: `tb_cmd_tau_new_project` → **immediately** `tb_cmd_tau_select_project` (same name) → `tb_cmd_tau_json_current` — **Project** must match intended name. **Never** skip `select project` after `new project` (global profile; not the open folder). Log in `tb_sys_context_log`. After multi-app resource creation, **`tau clear application`** + **`tau select project`** again so the user’s default `tau json current` shows the new project.

## Local execution / local builds (explicit only)

Do **not** automatically try to run, curl, open, or “test” Taubyte functions/libraries locally. Only load local execution or local verification workflows when the user **explicitly** asks to do so.

## Deep docs

`tb_sys_reference_index` — examples and extra topic routing.