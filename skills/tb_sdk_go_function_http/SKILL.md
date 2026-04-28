---
name: tb_sdk_go_function_http
description: Go Taubyte HTTP handlers: empty.go layout, Headers/Query/Return ordering.
---

## Layout (function scaffold)

- After `tau new function --template empty --language Go`, all hand-written Go lives in **root `empty.go`** next to `go.mod`.
- Keep `package lib` in authored files.
- Never hand-create `lib/`, never move `empty.go` under `lib/`, never hand-author `main.go` in the function tree.
- Keep `//export Handler` aligned with YAML `execution.call: Handler`.

## HTTP event API

- `h.Headers().Get(key)` and `h.Query().Get(key)` return `(string, error)`.
- Do not use `h.URL().Query()`; use `h.Query().Get(...)`.
- Set headers (e.g. `Content-Type`), **write body** (`h.Write`), then `h.Return(status)` **last**.

## Build/runtime alignment

- For remote/Dream project pushes, prefer `.taubyte/config.yaml` image `taubyte/go-wasi:latest`.
- For local-only Docker validation, use `tb_sdk_go_local_wasm_docker` recipe (`taubyte/go-wasi:v2`).

Global rules: `tb_sys_core_constraints`.