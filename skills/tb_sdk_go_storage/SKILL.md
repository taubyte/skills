---
name: tb_sdk_go_storage
description: Go storage SDK: Get/New, File Add/GetFile patterns.
---

## API

- `storage.Get(match)` or `storage.New(match)`
- Write: `stor.File(name).Add(data, overwrite)`
- Read: `sf, err := stor.File(name).GetFile()`; then `Read` / `io.Copy`; `defer sf.Close()`
- Do **not** use `stor.New().File(...)`

Rules: `tb_sys_core_constraints`.