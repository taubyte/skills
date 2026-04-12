# Taubyte go-sdk API — Must Use (avoid compile/build errors)

This file is the single source of truth for correct [github.com/taubyte/go-sdk](https://github.com/taubyte/go-sdk) usage. Deviations cause "assignment mismatch" or "undefined method" compile errors.

---

## HTTP event

| API                    | Return type       | Wrong                          | Right                                                          |
| ---------------------- | ----------------- | ------------------------------ | -------------------------------------------------------------- |
| `h.Headers().Get(key)` | `(string, error)` | `name := h.Headers().Get(...)` | `name, _ := h.Headers().Get(...)` or `name, err := ...`        |
| `h.Query().Get(key)`   | `(string, error)` | `name := h.Query().Get(...)`   | `name, _ := h.Query().Get(...)` or `name, err := ...`          |
| Query params           | —                 | `h.URL().Query().Get(...)`     | `h.Query().Get(...)` — **there is no `h.URL()`** on HTTP event |

---

## PubSub event

| API            | Return type               | Wrong           | Right                                          |
| -------------- | ------------------------- | --------------- | ---------------------------------------------- |
| `ev.Data()`    | `([]byte, error)`         | `_ = ev.Data()` | `data, err := ev.Data()` or `_, _ = ev.Data()` |
| `ev.Channel()` | `(*ChannelObject, error)` | single variable | use two variables                              |

---

## Storage

| Operation   | Wrong                                                                                   | Right                                                                                                                                                                                                       |
| ----------- | --------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Get storage | —                                                                                       | `stor, err := storage.Get(match)` then if needed `stor, err = storage.New(match)` (package-level `New`, not `stor.New()`)                                                                                   |
| Write file  | `stor.New().File(name)` then `content.Write` / `content.Push()`                         | **No `stor.New().File()`.** Use `stor.File(fileName).Add(data []byte, overWrite bool) (int, error)`.                                                                                                        |
| Read file   | `file, err := stor.File(name)` then `file.Close()` / `file.Read()` / `io.Copy(_, file)` | `stor.File(name)` returns `*File` (one value). `*File` has no `Read`/`Close`. Use `file := stor.File(name)`; `sf, err := file.GetFile()`; then `defer sf.Close()` and `io.Copy(dst, sf)` or `sf.Read(buf)`. |

---

## Quick copy-paste

**HTTP header/query (two return values):**

```go
name, _ := h.Headers().Get("X-File-Name")
if name == "" {
    name, _ = h.Query().Get("name")
}
```

**PubSub data (two return values):**

```go
data, err := ev.Data()
if err != nil {
    return 1
}
```

**Storage write:**

```go
body, _ := io.ReadAll(h.Body())
_, err := stor.File("filename.txt").Add(body, true)
```

**Storage read:**

```go
file := stor.File("filename.txt")
sf, err := file.GetFile()
if err != nil {
    return 1
}
defer sf.Close()
_, err = io.Copy(h, sf)
```

---

## Reference

- SDK source: [github.com/taubyte/go-sdk](https://github.com/taubyte/go-sdk) — `http/event/header.go`, `http/event/query.go`, `pubsub/event/event.go`, `storage/storage.go`, `storage/file.go`
- Detailed docs: `sdk-operation-http.md`, `sdk-operation-storage.md`, `sdk-function-pubsub.md`
