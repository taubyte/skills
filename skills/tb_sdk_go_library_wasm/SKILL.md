---
name: tb_sdk_go_library_wasm
description: Go WASM library repos: //export, execution.call, source.path, multi-route functions.
---

## Library repos

- Author Go at the library root (`empty.go` + `go.mod`) using `package lib`.
- **`//export <Symbol>`**; config `functions/*.yaml`: `source: libraries/<library>`, `execution.call: <Symbol>`.
- `libraries/*.yaml` **`source.path`** must match the library repo layout.

## Wiring checklist

1. `functions/<name>.yaml` has `source: libraries/<library_name>`.
2. `execution.call` equals exported symbol exactly.
3. Exported symbol exists once in code (`//export <Symbol>`).
4. Build image in `.taubyte/config.yaml` matches target path (`latest` for cloud, `v2` Docker for local verification).

## Common breakages

- Mixed `package main` / `package lib` across same module.
- Nested `lib/` source trees committed by hand.
- `execution.call` mismatch with export symbol.

Cross-refs: `tb_sys_resource_creation`, `tb_sys_reference_index`, `tb_sdk_go_function_http`, `tb_sdk_go_database`.