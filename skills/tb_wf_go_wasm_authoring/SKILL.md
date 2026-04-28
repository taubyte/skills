---
name: tb_wf_go_wasm_authoring
description: Workflow: load SDK split skills for HTTP/pubsub/storage/database/library/local verify.
---

Policy: `tb_sys_core_constraints`. After any selection change: `tb_sys_context_log`.

## Ordered skills

1. `tb_sys_core_constraints`
2. `tb_sdk_go_function_http` (function scaffold + HTTP handler order)
3. `tb_sdk_go_library_wasm` (library export/call wiring)
4. `tb_sdk_go_database`
5. `tb_sdk_go_pubsub`
6. `tb_sdk_go_storage`
7. `tb_sdk_go_local_wasm_docker`

## Canonical authoring flow

1. Scaffold function/library resources.
2. Keep authored Go at resource root (`empty.go` + `go.mod`), `package lib`, `//export <Symbol>`.
3. Ensure function YAML `execution.call` matches export symbol.
4. Align `.taubyte/config.yaml` image to cloud build target (`taubyte/go-wasi:latest` for remote/Dream project pushes).
5. Run local Docker WASM verification (`tb_sdk_go_local_wasm_docker`) when needed before code push.
6. Push config then code (`tb_wf_push_config_then_code`).

## Anti-patterns (do not do)

- Do not create `lib/main.go` or mix `package main` and `package lib`.
- Do not move `empty.go` under `lib/`.
- Do not trust a missing `artifact.wasm` as a packaging-only issue; fix source layout first.