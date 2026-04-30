---
name: writing-taubyte-functions
description: Authoring guide for the Taubyte Go SDK (`github.com/taubyte/go-sdk`) — covers the function signature, `event.Event` dispatch (HTTP / PubSub / P2P), and the SDK packages for HTTP request/response, outbound HTTP client, KV database, object storage, PubSub publish/subscribe + WebSocket URLs, P2P services, and IPFS files; includes the cross-library `//go:wasmimport` pattern, the YAML-to-Go matcher consistency rule, the body-then-Return ordering rule, and a complete handler skeleton. Use when writing or reviewing any Go function/library source for a Taubyte serverless workload.
---

# Writing Taubyte Functions (Go SDK)

## When to use

- Authoring a new Go handler (HTTP / PubSub / P2P)
- Reading or fixing existing function source
- Wiring a function to a database, storage, messaging channel, or library
- Calling another library's exported function from inside a function
- Calling external HTTP services from inside a function
- Producing PubSub messages from an HTTP function
- Surfacing a WebSocket URL to a browser
- Using IPFS-backed files

## Mental model

A Taubyte serverless function is a **TinyGo-compiled WebAssembly module**. The SDK (`github.com/taubyte/go-sdk`) is a Go wrapper over **host functions** the Taubyte VM exports. You write Go, compile to WASM (via the `taubyte/go-wasi` Docker recipe — see [verifying-taubyte-functions](../verifying-taubyte-functions/SKILL.md)), and the cloud invokes your exported function for each event.

Key constraints:

- **Package name is NOT `main`** — `main` is reserved for the build container. Use any other name (commonly `lib`).
- **Each handler is exported** with `//export <name>` and matches `execution.call` in the function's YAML.
- **Signature** is `func Handler(e event.Event) uint32` — returns `0` on success, non-zero on failure.
- **`event.Event`** is a tagged union; you call `.HTTP()`, `.PubSub()`, or `.P2P()` to get the typed handle, depending on the function's `trigger.type`.

**Anti-pattern:** switching the handler file to **`package main`** to "fix" compile errors when the real problem is **build hygiene** (generated `lib/`, `main.go`, bad `PATH` in `.taubyte/build.sh`, etc.). Stay on **`package lib`** (or another non-`main` name) per the constraints above; fix the toolchain and repo state instead — see [verifying-taubyte-functions](../verifying-taubyte-functions/SKILL.md) and [configuring-taubyte-build-runtime](../configuring-taubyte-build-runtime/SKILL.md).

## Module + imports

`go.mod` should depend on:

```text
github.com/taubyte/go-sdk
```

Common imports by feature:

```go
import (
    "github.com/taubyte/go-sdk/event"
    http       "github.com/taubyte/go-sdk/http/event"   // inbound HTTP
    httpclient "github.com/taubyte/go-sdk/http/client"  // outbound HTTP
    "github.com/taubyte/go-sdk/database"                // KV
    "github.com/taubyte/go-sdk/storage"                 // object storage
    pubsubnode "github.com/taubyte/go-sdk/pubsub/node"  // PubSub producer
    "github.com/taubyte/go-sdk/pubsub"                  // (consumer types)
    p2pnode    "github.com/taubyte/go-sdk/p2p/node"     // P2P producer/service
    "github.com/taubyte/go-sdk/ipfs/client"             // IPFS files
)
```

## Handler skeleton (HTTP)

```go
package lib

import (
    "github.com/taubyte/go-sdk/event"
    http "github.com/taubyte/go-sdk/http/event"
)

func fail(h http.Event, err error, code int) uint32 {
    h.Write([]byte(err.Error()))
    h.Return(code)
    return 1
}

//export Hello
func Hello(e event.Event) uint32 {
    h, err := e.HTTP()
    if err != nil {
        return 1
    }
    h.Headers().Set("Content-Type", "text/plain")
    if _, err := h.Write([]byte("hello")); err != nil {
        return fail(h, err, 500)
    }
    h.Return(200)
    return 0
}
```

The `//export Hello` symbol must match `execution.call: Hello` in the function YAML.

## Body-then-Return rule (critical)

For HTTP handlers, **always** in this order:

1. Set headers (`h.Headers().Set(...)`)
2. Write body (`h.Write(...)`)
3. Call `h.Return(<status>)` **last**

Reversing this — calling `Return` before `Write` — produces empty/broken responses; clients see a "the API does nothing" symptom. This is one of the most common Go-handler footguns.

## YAML ↔ Go matcher consistency rule

Every resource that uses a `match` (database, storage, messaging) is keyed at runtime by the **exact same string** the Go code passes to `New(...)` / `Channel(...)`. Mismatches (including a leading `/`) cause silent "open failed" / consumer never fires.

| Config YAML | Go literal |
| --- | --- |
| `databases/<x>.yaml` → `match: /example/kv` | `database.New("/example/kv")` |
| `storages/<x>.yaml` → `match: /simple/storage` | `storage.New("/simple/storage")` |
| `messaging/<x>.yaml` → `channel.match: chat` + PubSub fn `trigger.channel: chat` | `pubsubnode.Channel("chat")` |

Grep `database.New(`, `storage.New(`, `pubsubnode.Channel(` after every YAML edit and confirm each literal exists as a `match` in the same application.

## HTTP event API (`http/event`)

`HttpEvent` (received from `e.HTTP()`):

| Call | Purpose |
| --- | --- |
| `Body()` → `HttpEventBody` (Read/Close) | Request body reader |
| `Headers()` → `HttpEventHeaders` (Get/Set/List) | Request/response headers |
| `Host()` → `(string, error)` | Request host |
| `Method()` → `(string, error)` | HTTP method |
| `Path()` → `(string, error)` | Request path |
| `Query()` → `HttpQueries` (Get/List) | Query string |
| `UserAgent()` → `(string, error)` | UA header |
| `Write(data []byte)` → `(int, error)` | Response body write |
| `Return(code int)` → `error` | Send status — call LAST |

Read JSON body:

```go
defer h.Body().Close()
var req MyReq
if err := json.NewDecoder(h.Body()).Decode(&req); err != nil {
    return fail(h, err, 400)
}
```

Read query var:

```go
key, err := h.Query().Get("key")
```

## Outbound HTTP client (`http/client`)

For calling external services from a function:

```go
req, err := httpclient.New(httpclient.Method("GET"), httpclient.Headers(map[string][]string{
    "Accept": {"application/json"},
}))
// req.Headers().Set(...), req.Body() to set body, req.Method().Set("POST")
resp, err := req.Do()
data, err := io.ReadAll(resp.Body())
```

(API uses option functions `Method(...)`, `Headers(...)`, `Body(...)` and accessor methods `Headers()`, `Method()`, `Body()` for mutation after construction.)

## Database (`database`) — KV

```go
db, err := database.New("/example/kv")    // match exactly the YAML "match"
if err != nil { return fail(h, err, 500) }

err = db.Put("hello", []byte("world"))    // write
val, err := db.Get("hello")               // read -> []byte
keys, err := db.List("prefix/")           // list keys with prefix
err = db.Delete("hello")                  // delete
err = db.Close()                          // close (optional)
```

Taubyte databases are **instantiated on demand** when first opened. With a regex `match` like `/profile/history/[^/]+`, opening `/profile/history/userA` creates a per-user database transparently.

## Storage (`storage`) — object bucket

```go
sto, err := storage.New("/simple/storage")           // match exactly the YAML "match"
file := sto.File(filename)                            // select by name

// Write (overwrite=true to replace existing)
_, err = file.Add([]byte(data), true)

// Read
reader, err := file.GetFile()                         // io.ReadCloser
defer reader.Close()
io.Copy(h, reader)                                    // stream into HTTP response

// Versions / delete
versions, _ := file.Versions()
_ = file.Delete(version)
_ = file.DeleteAllVersions()

// Bucket-level
files, _ := sto.ListFiles()
remaining, _ := sto.RemainingCapacity()
```

## PubSub (`pubsub`, `pubsub/node`)

### Consumer (function with `trigger.type: pubsub`)

```go
//export OnChat
func OnChat(e event.Event) uint32 {
    ps, err := e.PubSub()
    if err != nil { return 1 }
    data, _ := ps.Data()                // []byte payload
    ch, _ := ps.Channel()               // *ChannelObject
    name := ch.Name()
    _ = name; _ = data
    return 0
}
```

In the function YAML:

```yaml
trigger:
  type: pubsub
  channel: chat            # must equal messaging channel.match AND pubsubnode.Channel("chat")
execution:
  call: OnChat
```

### Producer (called from any function — typically an HTTP handler)

```go
import pubsubnode "github.com/taubyte/go-sdk/pubsub/node"

ch, err := pubsubnode.Channel("chat")    // matches messaging channel.match
err = ch.Publish([]byte(payload))
```

### WebSocket URL (browser handoff)

```go
ch, err := pubsubnode.Channel("chat")
ws, err := ch.WebSocket()
u, err := ws.Url()                       // url.URL — return to client
h.Write([]byte(u.String()))
```

Requires `bridges.websocket.enable: true` in the messaging YAML.

### YAML wiring (must all match)

```
messaging.<x>.yaml: channel.match: chat
functions.<consumer>.yaml: trigger.channel: chat
Go consumer subscribes implicitly via trigger
Go producer: pubsubnode.Channel("chat")
```

## P2P (`p2p/event`, `p2p/node`)

### Consumer (function with `trigger.type: p2p`)

```go
//export OnP2P
func OnP2P(e event.Event) uint32 {
    p, err := e.P2P()
    if err != nil { return 1 }
    cmd, _   := p.Command()       // command name
    proto, _ := p.Protocol()      // protocol path
    data, _  := p.Data()          // payload
    p.Write([]byte("ack"))        // reply
    _ = cmd; _ = proto; _ = data
    return 0
}
```

Function YAML for a P2P handler:

```yaml
trigger:
  type: p2p
  protocol: /myproto/v1
  command: ping
execution:
  call: OnP2P
```

### Producer (call another node's P2P command)

```go
svc := p2pnode.New("/myproto/v1")
cmd, err := svc.Command("ping")
resp, err := cmd.Send([]byte("hello"))   // returns response bytes
```

A service that wants to listen:

```go
proto, err := svc.Listen()   // tells the node to accept this protocol
```

## IPFS files (`ipfs/client`)

```go
c, err := client.New()

// Write a new file → push to IPFS → get a CID
w, err := c.Create()
w.Write([]byte("hello ipfs"))
cidv, err := w.Push()        // returns cid.Cid

// Read by CID
r, err := c.Open(cidv)       // ReadOnlyContent (io.ReadSeekCloser + Cid())
data, err := io.ReadAll(r)
r.Close()
```

`ReadWriteContent` is `io.ReadWriteSeeker + io.Closer + Push() (cid.Cid, error)`.

## Importing another library's exported function

When a function lives in the code repo but needs to call code in a separate **library** repo, use `//go:wasmimport`:

```go
// libraries/<library_resource_name> is the source.path of the library YAML

//go:wasmimport libraries/tauhow_example_library add
func add(a, b uint32) uint64
```

Then use `add` like any Go function. The path before the symbol mirrors the library's `source.path` from `libraries/<x>.yaml`. Resolution is application-scoped first, then global.

The library itself just exports its function the normal way:

```go
package lib

//export add
func add(a, b uint32) uint64 {
    return uint64(a) + uint64(b)
}
```

## Function YAML cheatsheet (relevant fields)

```yaml
description: <one-liner>
source: .                              # ".": inline (code repo); else "libraries/<name>"
trigger:
  type: https | http | pubsub | p2p
  method: GET | POST | PUT | DELETE    # http(s) only
  paths: [ /api/x ]                    # http(s) only
  channel: <channel>                   # pubsub only
  protocol: <proto>                    # p2p only
  command: <cmd>                       # p2p only
domains: [ <domain_name> ]             # http(s) only
execution:
  timeout: 30s
  memory: 64MB
  call: <ExportedSymbol>               # MUST equal //export <name>
```

## Required project plumbing

- The function YAML's `execution.call` exactly matches a `//export <name>` symbol.
- Function root layout (Go): `go.mod` + source `.go` files at the function root. **Never** create a `lib/` subdirectory or hand-author `main.go`.
- For libraries that another function imports via `//go:wasmimport`, the `source.path` in `libraries/<x>.yaml` must reflect the actual on-disk path inside the library repo where the sources live.
- For storage/database/messaging matchers: see the YAML ↔ Go consistency table above.

## Workflow checklist

```
Function authoring progress:
- [ ] Resource YAML created via tau new function (creating-taubyte-resources)
- [ ] Package name is NOT `main`
- [ ] Each handler has //export <name> matching execution.call
- [ ] HTTP handlers: Headers().Set -> Write -> Return(status), in that order
- [ ] database.New / storage.New / pubsubnode.Channel literals match YAML "match"/"channel" exactly
- [ ] If using //go:wasmimport, library source.path matches the library repo layout
- [ ] Local WASM verify (verifying-taubyte-functions Docker recipe), if requested
- [ ] git push code (and library/website if applicable)
- [ ] Dream: dream inject push-specific for the changed repo (triggering-dream-builds)
```

## Gotchas

- **Empty body on success**: `Return(200)` was called before `Write([]byte(...))`. Reorder.
- **`database.New("...") error`**: matcher mismatch with `databases/<x>.yaml` — including a stray leading `/`.
- **PubSub consumer never fires**: `trigger.channel` ≠ messaging `channel.match` ≠ `pubsubnode.Channel("...")`. All three strings must be identical.
- **`WebSocket().Url()` returns empty**: messaging YAML missing `bridges.websocket.enable: true`.
- **Build error "package main not allowed"**: rename your package to anything other than `main` (commonly `lib`).
- **`//export` symbol not found in WASM**: typo between `//export` and YAML `execution.call`; or the symbol is unexported (lowercase first letter is fine — `//export` controls export, not Go's identifier case).
- **Library import path wrong**: `//go:wasmimport libraries/<name> <fn>` — `<name>` is the **library resource name** (matches the YAML filename / library identity), not a Go package import path.
- **Returning early on errors loses status**: write the error body and call `h.Return(code)` before the `return <non-zero>` (see the `fail` helper above).

## Reference docs

- Module: [github.com/taubyte/go-sdk](https://github.com/taubyte/go-sdk)
- Top-level docs: [tau.how](https://tau.how)
- Functions: [tau.how/development/functions](https://tau.how/development/functions/)
- Libraries: [tau.how/development/libraries](https://tau.how/development/libraries/)
- Databases: [tau.how/development/databases](https://tau.how/development/databases/)
- Storage: [tau.how/development/storage](https://tau.how/development/storage/)

## Related skills

- `creating-taubyte-resources` — `tau new function` flag patterns and YAML scaffolding
- `verifying-taubyte-functions` — local Go WASM build via the `taubyte/go-wasi` Docker recipe
- `building-taubyte-websites` — companion build rule (`/out`) for the website side
- `triggering-dream-builds` — push the function to Dream after editing
- `pushing-taubyte-projects` — push code/library/website repos
