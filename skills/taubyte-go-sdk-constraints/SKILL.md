---
name: taubyte-go-sdk-constraints
description: Must-use go-sdk API shapes to avoid Taubyte compile/build failures for HTTP, PubSub, and Storage functions.
---

# Go SDK Constraints

Use this skill for any Go function implementation or debugging.

## Go function filesystem layout (non-negotiable)

- **Authoring layout:** after `tau new function --template empty --language Go`, **all hand-written Go for the function lives in `empty.go` at the function root** — the same directory as `go.mod` (e.g. `code/functions/<name>/empty.go`). **Do not** put `empty.go` under `lib/` or any subdirectory for source you maintain.
- **Never manually create `lib/`**, never move or copy `empty.go` into `lib/`, and **never** hand-author **`main.go`** (or any extra `.go` shims) in the function tree to “fix” the build. That pattern duplicates packages, confuses the WASM layout, and breaks **`tau build function`** / CI (e.g. *found packages lib and main in `/src/lib`*).
- **Do not** “normalize” the tree: no merging a second `main.go` under `lib/`, no relocating the scaffold out of the root.
- If **`tau build function`** or a local experiment once dropped **`main.go`** or a **`lib/`** tree into the repo, treat those as **mistakes or stale artifacts** when they contradict the root-`empty.go` rule: **remove** stray `lib/` and hand-curated **`main.go`**, keep **only** root **`empty.go`** (+ `go.mod` / `.taubyte/*` as generated). Do not recreate `lib/` to satisfy the compiler.

## Local WASM build test (preferred over ad-hoc `tau build` tree hacks)

From the **function root** (directory containing `empty.go` and `go.mod`), with Docker available, use **`taubyte/go-wasi`** so the official `/utils/wasm.sh` path runs against a clean `/src` copy — **not** by adding `lib/` or `main.go` locally.

```bash
mkdir -p out
docker run -it --rm \
  -e CODE=/src \
  -v "$(pwd)/out:/out" \
  --mount type=bind,src="$(pwd)",dst=/src_ro,ro \
  --mount type=tmpfs,dst=/src \
  taubyte/go-wasi /bin/bash -c '
    set -e
    rsync -a --delete /src_ro/ /src/ 2>/dev/null || cp -a /src_ro/. /src/
    echo "export CODE=/src; cd /src; source /utils/wasm.sh" > /tmp/cowrc
    exec bash --rcfile /tmp/cowrc -i'
```

- **`CODE=/src` is required.** `/utils/wasm.sh` runs `go run . ${CODE}/lib lib`; if `CODE` is empty, `${CODE}/lib` becomes `/lib` and you get **package lib not found**.
- **`mv .git`** in `wasm.sh` may print *cannot stat '.git'* when there is no `.git` in the function folder; that is harmless.
- Run interactively inside the container, then run **`build ./`** (or your target path). Artifacts land under **`./out`** on the host (`artifact.wasm`, `_artifact.wasm`).
- **Non-interactive one-shot** (same mounts; no TTY): after `cd` to the function root, `source /utils/wasm.sh` then **`build ./`** in the same `bash -c` string (omit `set -e` if `mv .git` would abort your wrapper).
- On **Windows**, use **Git Bash** with **`MSYS_NO_PATHCONV=1`** in front of `docker` so paths like `/out` are not rewritten; use **`$(pwd -W)`** for bind mounts if Docker Desktop needs a Windows path.

## HTTP event constraints

- `h.Headers().Get(key)` returns `(string, error)` (two variables).
- `h.Query().Get(key)` returns `(string, error)` (two variables).
- Do not use `h.URL().Query()...`; use `h.Query().Get(...)`.
- **Response ordering:** set headers (including `Content-Type` for JSON), **write the body** (`h.Write(...)`), then call **`h.Return(status)`** last. Calling `Return` before `Write` yields empty or truncated bodies and broken `response.json()` clients.

## PubSub event constraints (consumer — `trigger.type: pubsub`)

- From the PubSub event object: `Data()` returns `([]byte, error)` (two variables); `Channel()` returns `(*ChannelObject, error)` (two variables). Typical pattern: `pubsubEvent, err := e.PubSub()` then `pubsubEvent.Data()` / `pubsubEvent.Channel()` — align with your handler’s generated event parameter name.
- Optionally verify `Channel().Name()` equals the configured channel when multiple channels exist.
- Deserialize `Data()`, validate, then optionally open KV with `database.New("<match>")` and `Put` / other side effects.

## PubSub producer and WebSocket (HTTP handlers — `pubsub/node`)

- `import pubsubnode "github.com/taubyte/go-sdk/pubsub/node"`.
- `ch, err := pubsubnode.Channel("<channelName>")` — the string must equal the **messaging** resource’s `channel.match` and any PubSub function’s `trigger.channel` in the same project wiring.
- `ch.Publish(payloadBytes)` to broadcast from an HTTP handler (check errors in real code).
- For browser realtime: `url, err := ch.WebSocket().Url()` and return `url.String()` (e.g. JSON) to the client; messaging YAML should have **`bridges.websocket.enable: true`** when using WebSocket URLs.

**Cross-wiring checklist:** `messaging.channel.match` == `trigger.channel` on PubSub functions == `pubsubnode.Channel("...")`; keep `messaging.local` and `trigger.local` aligned when both exist; `execution.call` must match `//export` in the built WASM.

## Storage constraints

- Retrieve storage via `storage.Get(match)` or `storage.New(match)`.
- Write with `stor.File(fileName).Add(data, overwrite)`.
- Read with:
  - `file := stor.File(fileName)`
  - `sf, err := file.GetFile()`
  - then `sf.Read(...)` or `io.Copy(...)` and `sf.Close()`.
- Do not use invalid patterns like `stor.New().File(...)`.

## Database (KV) — `github.com/taubyte/go-sdk/database`

- `import "github.com/taubyte/go-sdk/database"`.
- `db, err := database.New("<match>")` — **`<match>` must equal** the database resource’s YAML **`match`** when `useRegex: false` (not the file name or `description`). Path-style matchers are exact: e.g. YAML `match: /todos` requires `database.New("/todos")` including the leading `/`.
- `defer db.Close()` when the DB is opened for the scope of one request or event.
- **`Put(key, []byte)`**, **`Get(key)`** → `([]byte, error)`, **`Delete(key)`**, **`List("<prefix>")`** → keys under a prefix (e.g. list `todo/` then `Get` each). Sort in application code if you need stable ordering.
- **Missing keys:** `Get` failing or empty value is often **normal** (first run, unknown id). For list APIs, treat “no index key yet” as an **empty list**, not necessarily HTTP 500 — reserve 500 for real store errors.
- **Key design:** namespace with prefixes (`todo/<id>`, `note/<id>`, …). Prefer **one database resource per app** with a stable `match` and many prefixes unless policy requires splitting.

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

Database (import `"github.com/taubyte/go-sdk/database"`; `match` must match YAML):

```go
db, err := database.New("appdata")
if err != nil { return 1 }
defer db.Close()
_ = db.Put("todo/1", []byte(`{"title":"x"}`))
raw, err := db.Get("todo/1")
keys, err := db.List("todo/")
```

## Go WASM libraries (not the `empty.go` function scaffold)

**Library** resources are **separate repos**: they ship **`.taubyte/`** + **`go-sdk`** and expose handlers with **`//export <Symbol>`** comments. Config **`functions/*.yaml`** then sets **`source: libraries/<library>`** and **`execution.call: <Symbol>`** per route — one build, many HTTP resources (see **`taubyte-resource-creation`** and **`taubyte-reference-index`** → *Full-stack worked example*).

- The **“root `empty.go` only / never hand-create `lib/`”** rule applies to **`tau new function --template empty`** trees, **not** to authoring conventions inside an **external library repo** the platform builds as WASM.
- Use the **same HTTP and database API shapes** as above (`h.Query().Get`, `database.New` path matching the **`databases` `match`** in config, etc.).
- **`libraries/*.yaml`** **`source.path`** must match the **directory layout inside the library’s GitHub repo** that actually contains the built Go sources (often repo root `/`). Wrong `source.path` → wrong or empty WASM.
- Keep **response bodies** and **`Content-Type`** consistent with the **front-end** (e.g. JSON vs raw bytes) so **`response.json()`** matches what the handler writes; **write body then `Return`** (see HTTP constraints above).
