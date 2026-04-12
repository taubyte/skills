# Taubyte Storage Operations (SDK)

## CRITICAL — go-sdk v0.3.x API (must match exactly)

- **Storage has no `.New()` method.** `New` is package-level: `storage.New(match)`, not `stor.New()`.
- **There is no `stor.New().File(name)`.** To write: use `stor.File(fileName).Add(data []byte, overWrite bool) (int, error)`.
- **`stor.File(name)` returns a single value `*File`.** `*File` has no `Read()` or `Close()`. To read: call `file.GetFile()` → `(*StorageFile, error)`; then `*StorageFile` has `Read(p []byte)` and `Close()`. Use `io.Copy(dst, sf)` with `defer sf.Close()`.
- **Query/headers:** Use `h.Query().Get("key")` (not `h.URL().Query()` — HTTP event has no `URL()`). `Headers().Get` and `Query().Get` both return `(string, error)`; always assign to two variables.

Reference: [github.com/taubyte/go-sdk](https://github.com/taubyte/go-sdk) — `storage/storage.go`, `storage/file.go`, `http/event/header.go`, `http/event/query.go`.

## Matcher vs Resource Name

**Use the matcher (match pattern), NOT the resource name.** The SDK resolves storage by the `match` value from the storage config (`config/storage/[name].yaml`). The config file name is the resource name; the `match:` field is the matcher. In code, always pass the matcher to `storage.Get()` and `storage.New()`.

## Getting/Creating Storage

```go
import "github.com/taubyte/go-sdk/storage"

// Use the match value from config (e.g. match: uploads), NOT the resource name (e.g. my_storage)
stor, err := storage.Get("uploads")
if err != nil {
    // Create new storage - package-level New(), same matcher
    stor, err = storage.New("uploads")
    if err != nil {
        return 1
    }
}
```

## File Operations

### Write / overwrite file (Add)

**Use `stor.File(fileName).Add(data, overWrite)`.** There is no `stor.New().File()`.

```go
stor, _ := storage.Get("uploads")
data := []byte("file content")
_, err := stor.File("filename.txt").Add(data, true)  // true = overwrite
if err != nil {
    return 1
}
```

### Read file (GetFile → StorageFile)

**`stor.File(name)` returns `*File` (one value).** `*File` does not implement `io.Reader` and has no `Read`/`Close`. Call `file.GetFile()` to get `*StorageFile`, then use `Read`/`Close` or `io.Copy`.

```go
file := stor.File("filename.txt")
sf, err := file.GetFile()
if err != nil {
    return 1  // e.g. file not found
}
defer sf.Close()

// Option A: io.Copy to another writer
_, err = io.Copy(responseWriter, sf)

// Option B: read into buffer
buffer := make([]byte, 1024)
n, err := sf.Read(buffer)
```

### Delete File

```go
err = stor.File("filename.txt").Delete()
```

### List All Files

```go
files, err := stor.ListFiles()
```

## File Versioning

### Get File Versions

```go
versions, err := stor.File("filename.txt").Versions()
```

### Get Current Version

```go
currentVersion, err := stor.File("filename.txt").CurrentVersion()
```

## Complete Upload Example (correct API)

```go
package lib

import (
	"io"
	"github.com/taubyte/go-sdk/event"
	"github.com/taubyte/go-sdk/storage"
)

//export uploadFile
func uploadFile(e event.Event) uint32 {
	h, err := e.HTTP()
	if err != nil {
		return 1
	}

	stor, err := storage.Get("uploads")
	if err != nil {
		stor, err = storage.New("uploads")
		if err != nil {
			return 1
		}
	}

	fileData, err := io.ReadAll(h.Body())
	if err != nil {
		return 1
	}
	defer h.Body().Close()

	// Correct: use File(name).Add(data, overWrite), NOT stor.New().File()
	_, err = stor.File("uploaded_file.txt").Add(fileData, true)
	if err != nil {
		return 1
	}

	h.Return(200)
	return 0
}
```

## Complete Download Example (correct API)

```go
file := stor.File("filename.txt")
sf, err := file.GetFile()
if err != nil {
	h.Return(404)
	return 0
}
defer sf.Close()
_, err = io.Copy(h, sf)
if err != nil {
	h.Return(500)
	return 0
}
h.Return(200)
return 0
```

## Key Takeaways

1. **Matcher**: Use the **match** value from storage config in `storage.Get(match)` and `storage.New(match)`, NOT the resource name.
2. **Get or Create**: Use `storage.Get(match)` first, then `storage.New(match)` if not found (package-level, not `stor.New()`).
3. **Write**: Use `stor.File(fileName).Add(data, overWrite)` — no `stor.New().File()` or `Content.Push()` for simple uploads.
4. **Read**: Use `stor.File(name)` → `file.GetFile()` → `*StorageFile`; then `sf.Read()` / `sf.Close()` or `io.Copy(_, sf)`.
5. **Never**: Use `stor.File(name)` as an `io.Reader`, or call `Close()`/`Read()` on `*File` — only on `*StorageFile` from `GetFile()`.
