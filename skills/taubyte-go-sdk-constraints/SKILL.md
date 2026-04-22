---
name: taubyte-go-sdk-constraints
description: Must-use go-sdk API shapes to avoid Taubyte compile/build failures for HTTP, PubSub, and Storage functions.
---

# Go SDK Constraints

Use this skill for any Go function implementation or debugging.

## HTTP event constraints

- `h.Headers().Get(key)` returns `(string, error)` (two variables).
- `h.Query().Get(key)` returns `(string, error)` (two variables).
- Do not use `h.URL().Query()...`; use `h.Query().Get(...)`.

## PubSub event constraints

- `ev.Data()` returns `([]byte, error)` (two variables).
- `ev.Channel()` returns `(*ChannelObject, error)` (two variables).

## Storage constraints

- Retrieve storage via `storage.Get(match)` or `storage.New(match)`.
- Write with `stor.File(fileName).Add(data, overwrite)`.
- Read with:
  - `file := stor.File(fileName)`
  - `sf, err := file.GetFile()`
  - then `sf.Read(...)` or `io.Copy(...)` and `sf.Close()`.
- Do not use invalid patterns like `stor.New().File(...)`.

## Quick patterns

```go
name, _ := h.Headers().Get("X-File-Name")
name, _ = h.Query().Get("name")

data, err := ev.Data()
if err != nil { return 1 }

_, err = stor.File("filename.txt").Add(body, true)
file := stor.File("filename.txt")
sf, err := file.GetFile()
defer sf.Close()
```
