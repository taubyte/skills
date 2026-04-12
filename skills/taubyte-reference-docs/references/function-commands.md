# Function Commands

## Create Function

```bash
tau new function [flags] <name>
```

Flags:

- `--name`, `-n`: Function name (can also be provided as positional argument)
- `--description`, `-d`: Function description
- `--timeout`, `--ttl`: Time to live for an instance (**CRITICAL**: NEVER set to `0` - it will cause an error. Use valid duration format like `30s`, `1m`, `5m` or omit to use defaults)
- `--memory`, `--me`: Max memory either in form 10GB or 10
- `--memory-unit`, `--mu`: Unit if not provided with memory
- `--type`: Function type (http, p2p, pubsub)
- `--source`: Path within the code folder or a library repository
- `--call`, `--ca`: Exported function to call
- `--template`: Template URL or name
- `--language`, `--lang`: Template language
- `--use-template`: Use template
- `--yes`, `-y`: Skip confirmation prompts

**HTTP Function Flags:**

- `--method`, `-m`: HTTP method (GET, POST, PUT, DELETE, etc.)
- `--domains`: List of domains (comma separated) by name or FQDN
- `--paths`, `-p`: HTTP paths to use for the endpoint. **CRITICAL**: Paths should be without quotes. **CRITICAL (Windows)**: On Windows (Git Bash), prefix the command with `MSYS_NO_PATHCONV=1` to prevent path conversion: `MSYS_NO_PATHCONV=1 tau new function --paths / ...`
- `--generate-domain`, `-g`: Generates a new domain if no domains found

**P2P Function Flags:**

- `--protocol`, `--pr`: Protocol to use for the endpoint
- `--command`, `--cmd`: Command to execute
- `--local`, `-l`: Enable local execution

**PubSub Function Flags:**

- `--channel`, `--ch`: Channel to subscribe to
- `--local`, `-l`: Enable local execution

**Interactive Prompts:**

- Use Template: True/False (can be bypassed with --use-template)
- Description: (can be bypassed with --description)
- Time To Live: (can be bypassed with --timeout)
- Memory: (can be bypassed with --memory)
- Type: (can be bypassed with --type)
- Domains: (can be bypassed with --domains)
- Method: (can be bypassed with --method)
- Paths: (can be bypassed with --paths)
- Code source: (can be bypassed with --source)
- Call: (can be bypassed with --call)
- Confirm creation: Yes/No (can be bypassed with --yes)

Examples:

```bash
# Non-interactive mode (ALWAYS use these flags):
# HTTP Function:
tau new function --description "API handler function" --type http --method POST --paths /api/users --domains api.example.com --timeout 5s --memory 10MB --source . --call api-handler --template empty --language Go --defaults --yes api-handler

# PubSub Function:
tau new function --description "PubSub handler function" --type pubsub --channel events --timeout 5s --memory 10MB --source . --call pubsub-handler --template empty --language Go --defaults --yes pubsub-handler

# P2P Function:
tau new function --description "P2P handler function" --type p2p --protocol substrate --command process --timeout 5s --memory 10MB --source . --call p2p-handler --template empty --language Go --defaults --yes p2p-handler

# Function using a library:
tau new function --description "Function using library" --type http --method GET --paths /api/data --domains api.example.com --timeout 5s --memory 10MB --source libraries/mylib --call myFunction --template empty --language Go --defaults --yes my-function

# Interactive mode (will prompt):
tau new function
tau new function --type http api-handler
tau new function \
  --type http \
  --method POST \
  --paths /api/users \
  --domains api.example.com \
  api-handler
# Note: Paths without quotes.
tau new function --type p2p p2p-handler
tau new function --type pubsub --channel events pubsub-handler
```

## Query Function

```bash
tau query function [flags] <name>
```

Flags:

- `--name`, `-n`: Function name (can also be provided as positional argument)
- `--list`: List all functions instead of querying
- `--select`: Force selection prompt

Examples:

```bash
tau query function
tau query function api-handler
tau query function --list
```

## List Functions

```bash
tau list functions
```

Aliases:

- `tau list function`

Examples:

```bash
tau list functions
```

## Edit Function

```bash
tau edit function <name>
```

Examples:

```bash
tau edit function
tau edit function api-handler
```

## Delete Function

```bash
tau delete function <name>
```

Flags:

- `--yes`, `-y`: Skip confirmation prompts

Examples:

```bash
tau delete function
tau delete function api-handler
```

## Non-Interactive Mode

**CRITICAL**: For non-interactive mode, ALWAYS include ALL required flags based on function type:

**HTTP Function:**

```bash
tau new function --description "[description]" --type http --method [METHOD] --domains [domain] --paths [path] --timeout 5s --memory 10MB --source . --call [functionName] --template empty --language Go --defaults --yes <name>
```

**CRITICAL**:

- For `--paths`, do NOT use quotes.
- **CRITICAL (Windows)**: On Windows (Git Bash), prefix the command with `MSYS_NO_PATHCONV=1` to prevent path conversion: `MSYS_NO_PATHCONV=1 tau new function --paths / ...`
- **MUST** include `--template empty --language Go` to get Go templates (defaults to Rust without `--language Go`)
- `--source` and `--call` are REQUIRED when using `--defaults` flag
- **`--source` usage:**
  - If NOT using a library: Use `--source .` (dot means current function directory)
  - If using a library: Use `--source libraryfile` (path to library file)
- **`--call` usage:** Always use the function name (e.g., `getNotes`, `saveNote`)
- The empty template creates `empty.go` file that needs to be renamed to match function name (e.g., `[functionName].go`)

**PubSub Function:**

```bash
tau new function --description "[description]" --type pubsub --channel [channel] --timeout 5s --memory 10MB --source . --call [functionName] --template empty --language Go --defaults --yes <name>
```

**Note:** `--source` and `--call` are REQUIRED when using `--defaults` flag. Use `--source .` when not using a library.

**P2P Function:**

```bash
tau new function --description "[description]" --type p2p --protocol [protocol] --command [command] --timeout 5s --memory 10MB --source . --call [functionName] --template empty --language Go --defaults --yes <name>
```

**Note:** `--source` and `--call` are REQUIRED when using `--defaults` flag. Use `--source .` when not using a library.

**Required flags (all types):**

- `--description`: Function description (use empty string `""` if not needed)
- `--type`: Function type (`http`, `pubsub`, or `p2p`)
- `--timeout`: Time to live (e.g., `5s`, `30s`, `1m`) - NEVER set to `0`
- `--memory`: Max memory (e.g., `10MB`, `256MB`)
- `--source`:
  - If NOT using a library: Use `.` (dot means current function directory)
  - If using a library: Use `libraryfile` (path to library file, e.g., `libraries/mylib`)
- `--call`: Function name to call (must match exported function name in code, e.g., `getNotes`, `saveNote`)
- `--template empty`: Use empty template
- `--language Go`: Specify Go language (REQUIRED for Go functions - defaults to Rust without this)
- `--yes`: Skip confirmation prompts

**Required flags (HTTP):**

- `--method`: HTTP method(s) (e.g., `POST`, `GET,POST`)
- `--domains`: Comma-separated list of domain names
- `--paths`: Comma-separated list of HTTP paths (NO QUOTES).

**Required flags (PubSub):**

- `--channel`: Channel name to subscribe to

**Required flags (P2P):**

- `--protocol`: Protocol to use (e.g., `substrate`)
- `--command`: Command to execute

**Template File Structure:**

- The empty template with `--language Go` creates:
  - `empty.go` - Rename this to `[functionName].go` and update the function name in the `//export` comment
  - `go.mod` - Module file (update module name if needed)
  - `.taubyte/config.yaml` - Build configuration (already correct for Go)
  - `.taubyte/build.sh` - Build script (already correct)
- After creation, rename `empty.go` to match your function name and update the exported function name

**Source and Call Flag Usage:**

- **`--source` flag:**
  - If NOT using a library: Use `.` (dot means current function directory)
  - If using a library: Use `libraryfile` (path to library file, e.g., `libraries/mylib`)
- **`--call` flag:**
  - Always use the function name (e.g., `getNotes`, `saveNote`, `deleteNote`)
  - Must match the exported function name in your code (the name after `//export`)

## Common Patterns

### Creating HTTP API Endpoint (Non-Interactive)

```bash
tau new function \
  --description "User API endpoint" \
  \
  --type http \
  --method POST \
  --domains api.example.com \
  --paths /api/users \
  --timeout 30s \
  --memory 256MB \
  --source . \
  --call user-api \
  --template empty \
  --language Go \
  --defaults --yes \
  user-api
```

### Creating P2P Command Handler (Non-Interactive)

```bash
tau new function \
  --description "Data processor function" \
  \
  --type p2p \
  --protocol substrate \
  --command process \
  --local \
  --timeout 60s \
  --memory 10MB \
  --source . \
  --call data-processor \
  --template empty \
  --language Go \
  --defaults --yes \
  data-processor
```

### Creating Event Handler (Non-Interactive)

```bash
tau new function \
  --description "Event processor function" \
  \
  --type pubsub \
  --channel events \
  --local \
  --timeout 30s \
  --memory 10MB \
  --source . \
  --call event-processor \
  --template empty \
  --language Go \
  --defaults --yes \
  event-processor
```

### Function with Custom Configuration (Non-Interactive)

```bash
tau new function \
  --description "Complex API handler" \
  \
  --type http \
  --method GET,POST \
  --domains api.example.com,api2.example.com \
  --paths /api/v1,/api/v2 \
  --timeout 45s \
  --memory 1024 \
  --memory-unit MB \
  --source . \
  --call complex-handler \
  --template empty \
  --language Go \
  --defaults --yes \
  complex-handler
```

## Advanced Usage

### Multiple HTTP Methods (Non-Interactive)

```bash
tau new function \
  --description "API handler function" \
  \
  --type http \
  --method GET,POST,PUT,DELETE \
  --domains api.example.com \
  --paths /api/resource \
  --timeout 5s \
  --memory 10MB \
  --source . \
  --call api-handler \
  --template empty \
  --language Go \
  --defaults --yes \
  api-handler
```

### Multiple Domains and Paths (Non-Interactive)

```bash
tau new function \
  --description "Multi-domain function" \
  \
  --type http \
  --method GET \
  --domains api1.example.com,api2.example.com \
  --paths /v1,/v2 \
  --timeout 5s \
  --memory 10MB \
  --source . \
  --call multi-domain \
  --template empty \
  --language Go \
  --defaults --yes \
  multi-domain
```

### Function Using a Library (Non-Interactive)

```bash
tau new function \
  --description "Function using shared library" \
  \
  --type http \
  --method GET \
  --domains api.example.com \
  --paths /api/data \
  --timeout 5s \
  --memory 10MB \
  --source libraries/mylib \
  --call myFunction \
  --template empty \
  --language Go \
  --defaults --yes \
  my-function
```

**Note:** When using a library, `--source` should point to the library path (e.g., `libraries/mylib`), and `--call` should be the function name exported from that library.

### Function with Call URL (Non-Interactive)

```bash
tau new function \
  --description "Proxy function" \
  \
  --type http \
  --method GET \
  --domains api.example.com \
  --paths /proxy \
  --call https://external-api.com/endpoint \
  --defaults --yes \
  proxy-function
```

### Local Execution (Non-Interactive)

```bash
tau new function \
  --description "Local processor function" \
  \
  --type p2p \
  --protocol substrate \
  --command local-process \
  --local \
  --defaults --yes \
  local-processor
tau new function \
  --description "Local events function" \
  \
  --type pubsub \
  --channel local-events \
  --local \
  --defaults --yes \
  local-events
```

## Troubleshooting

### Error: "Invalid function type"

- Valid types are: `http`, `p2p`, `pubsub`
- Check spelling and case (lowercase required)

### Error: "Function name already exists"

- Use a different name
- Or edit the existing function instead
- Check existing functions: `tau list functions`

### Error: "Invalid HTTP method"

- Valid methods: GET, POST, PUT, DELETE, PATCH, HEAD, OPTIONS
- Use a **single** method per function. For multiple methods on the same path, create **separate functions** (e.g. get_notes with `--method GET`, save_note with `--method POST`, delete_note with `--method DELETE`).
- Check method spelling (uppercase: GET, POST, etc.).

### Error: Config build fails for HTTP function (multi-method)

- **Cause:** Using comma-separated `--method GET,POST,DELETE` produces invalid config; the config build pipeline does not accept multiple methods on one function.
- **Fix:** Create **one function per (path + method)**. Example: for `/api/notes` with GET, POST, DELETE, create three functions—e.g. `get_notes` (GET), `save_note` (POST), `delete_note` (DELETE)—each with a single `--method`. Never use `--method GET,POST,DELETE`.

### Error: "Domain not found"

```bash
tau new domain --description "API domain" --generated-fqdn --defaults --yes api.example.com
tau new function --description "API handler function" --type http --method GET --domains api.example.com --paths /api --defaults --yes api-handler
# Note: Paths should be without quotes.
```

### Error: "Invalid timeout format"

- Use duration format: `30s`, `5m`, `1h`
- Examples: `30s`, `2m30s`, `1h`
- **CRITICAL**: NEVER set `--timeout` to `0` - it will cause an error. Always use a valid duration or omit the flag to use defaults.

### Error: "path `C:/Program Files/Git/api/notes` is not valid, should start with /" or path parsing issues

**Problem**: On Windows (Git Bash), Git Bash/MSYS automatically converts Unix-style paths that start with `/` to Windows file system paths (e.g., `C:/Program Files/Git/...`), causing path parsing errors.

**Solution**:

- Prefix the command with `MSYS_NO_PATHCONV=1` to prevent path conversion: `MSYS_NO_PATHCONV=1 tau new function --paths /api/notes ...`
- Do NOT use quotes around paths: Use `--paths /api/notes` NOT `--paths "/api/notes"`
- For single root path `/`, use `--paths /`
- For multiple paths, use `--paths /path1,/path2` format (comma-separated, no quotes)
- Examples:
  - Single path: `MSYS_NO_PATHCONV=1 tau new function --paths /api/notes ...`
  - Root path: `MSYS_NO_PATHCONV=1 tau new function --paths / ...`
  - Multiple paths: `MSYS_NO_PATHCONV=1 tau new function --paths /api/notes,/api/users,/api/posts ...`
- **Alternative (persistent)**: Set `export MSYS2_ARG_CONV_EXCL='--paths'` in your `.bashrc` to exclude the `--paths` flag from conversion permanently

### Error: "Invalid memory specification"

- Use format: `--memory 512 --memory-unit MB`
- Or: `--memory 512MB`
- Valid units: MB, GB

### Error: "assignment mismatch: 1 variable but ... returns 2 values" (Headers, Query, or PubSub Data)

- **Cause:** In go-sdk, `Headers().Get()`, `Query().Get()`, and PubSub `Data()` return two values `(value, error)`. Using a single variable causes this compile error.
- **Fix:** Always assign to two variables, e.g. `name, _ := h.Headers().Get("X-Name")`, `q, _ := h.Query().Get("name")`, `data, err := ev.Data()`. Use `h.Query().Get(...)` for query params (there is no `h.URL()`). See `sdk-api-must.md`.

### Error: "no taubyte directory found in .../showcase_app/functions/event_handler"

- **Cause:** The build pipeline expects function code at `<app_name>/functions/<function_name>/` at the **root** of the code repo, not under `applications/`. If `tau new function` created `code/applications/<app_name>/functions/<name>/`, the builder looks for `code/<app_name>/functions/<name>/` and does not find `.taubyte/`.
- **Fix:** Move the function directories from `code/applications/<app_name>/functions/` to `code/<app_name>/functions/` at the repo root (e.g. move `applications/showcase_app/functions/*` to `showcase_app/functions/`), then remove the empty `applications/` directory. Ensure each function folder contains `.taubyte/config.yaml` and `.taubyte/build.sh`. See `taubyte-folder-examples.md` (Code Repository Layout).

### Error: storage "undefined ... New" or "File has no field or method Close" or "does not implement io.Reader"

- **Cause:** go-sdk storage API: `Storage` has no `.New()` method (use package `storage.New(match)`). `stor.File(name)` returns `*File`; `*File` has no `Read`/`Close` — use `file.GetFile()` to get `*StorageFile` for read/close. To write, use `stor.File(name).Add(data, true)`, not `stor.New().File(name)`.
- **Fix:** See `sdk-api-must.md` and `sdk-operation-storage.md` (CRITICAL section).

### Remote build: "package lib not found" then SIGSEGV in TinyGo optimizer

- **Symptom:** Build logs show `package lib not found` / `exit status 1`, then later `fatal error: unexpected signal ... SIGSEGV` in `tinygo.org/x/go-llvm` (e.g. `LLVMRunFunctionPassManager`). Subsequent functions may fail with "artifact.wasm: The system cannot find the file specified" because the job failed earlier.
- **Cause:** TinyGo/LLVM optimizer can crash on some code (e.g. heavy `map[string]interface{}`, `[]map[string]string`, or complex reflection).
- **Fix:** Simplify the failing function: use **concrete structs** with JSON tags and `[]Struct` instead of `[]map[string]string` or `map[string]interface{}` for JSON responses. See `build-test-functions-docker.md` (TinyGo optimizer crash). To fetch full logs: `check-logs-debug.md`.

### Function Not Triggering

```bash
tau query function <name>
tau list domains
tau query builds
```
