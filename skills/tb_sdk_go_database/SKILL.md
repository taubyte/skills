---
name: tb_sdk_go_database
description: Go database KV: database.New match vs YAML, Get/Put/List, defer Close.
---

## Import

`import "github.com/taubyte/go-sdk/database"`

## Rules

- `database.New("<match>")` must equal YAML **`match`** when `useRegex: false`.
- `defer db.Close()` for request/event scope.

## Minimal operations

```go
db, err := database.New("/todos")
if err != nil {
    return 1
}
defer db.Close()

if err := db.Put("k1", []byte("v1")); err != nil {
    return 1
}

val, err := db.Get("k1")
if err != nil {
    return 1
}
_ = val

items, err := db.List()
if err != nil {
    return 1
}
_ = items
```

## Alignment checks

- Compare all `database.New(...)` literals against `config/applications/<app>/databases/*.yaml` `match`.
- If `useRegex: true`, code literals must still satisfy the regex pattern expected by runtime routing.

Rules: `tb_sys_core_constraints`.