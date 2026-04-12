# Build and Test Functions Locally with Docker

Use the same image and build flow as CI so you catch compile and SDK errors **before** pushing. Run from each function directory.

## When to use

- After writing or changing function code (HTTP, PubSub, P2P).
- To verify SDK usage (e.g. two return values for Headers/Query/Data, storage GetFile/Add) compiles.
- To avoid failed remote builds and long feedback loops.

## One-shot build (non-interactive)

From the **function directory** (e.g. `code/<app_name>/functions/<function_name>/`):

```bash
cd /path/to/code/<app_name>/functions/<function_name>

# Linux/macOS
docker run --rm \
  -v "$(pwd)/out:/out" \
  --mount type=bind,src="$(pwd)",dst=/src_ro,ro \
  --mount type=tmpfs,dst=/src \
  taubyte/go-wasi /bin/bash -c '
    rsync -a --delete /src_ro/ /src/ 2>/dev/null || cp -a /src_ro/. /src/
    cd /src && source /utils/wasm.sh && export FILENAME=<function_name> && build "${FILENAME}"
    echo "BUILD_EXIT=$?"
  '
```

**Windows (Git Bash):** use `MSYS_NO_PATHCONV=1` so Docker mount paths are not converted (e.g. `/src` must stay `/src`, not `C:/Program Files/Git/src`):

```bash
cd /path/to/code/<app_name>/functions/<function_name>

MSYS_NO_PATHCONV=1 docker run --rm \
  -v "$(pwd)/out:/out" \
  --mount type=bind,src="$(pwd)",dst=/src_ro,ro \
  --mount type=tmpfs,dst=/src \
  taubyte/go-wasi /bin/bash -c '
    rsync -a --delete /src_ro/ /src/ 2>/dev/null || cp -a /src_ro/. /src/
    cd /src && source /utils/wasm.sh && export FILENAME=<function_name> && build "${FILENAME}"
    echo "BUILD_EXIT=$?"
  '
```

Replace `<function_name>` with the actual name (e.g. `health`, `get_file`, `event_handler`).

## Result

- **Success:** `BUILD_EXIT=0`. You may see `package lib not found` and `exit status 1` from the Go step; if the last line is `BUILD_EXIT=0` and the log shows `* Compile [DONE]` / `* Optimize [DONE]`, the build succeeded.
- **Failure:** `BUILD_EXIT=1` and compile errors in the log (e.g. assignment mismatch, undefined method). Fix the code using `sdk-api-must.md` and `sdk-operation-*.md`, then re-run.

## Interactive shell (optional)

To get a shell inside the container and run builds manually:

```bash
cd /path/to/code/<app_name>/functions/<function_name>

# Linux/macOS (use -it for TTY)
docker run -it --rm \
  -v "$(pwd)/out:/out" \
  --mount type=bind,src="$(pwd)",dst=/src_ro,ro \
  --mount type=tmpfs,dst=/src \
  taubyte/go-wasi /bin/bash -c '
    set -e
    rsync -a --delete /src_ro/ /src/ 2>/dev/null || cp -a /src_ro/. /src/
    echo "cd /src; source /utils/wasm.sh" > /tmp/cowrc
    exec bash --rcfile /tmp/cowrc -i
  '
```

Then inside the container:

```bash
export FILENAME=health   # or get_file, event_handler, etc.
build "${FILENAME}"
```

On Windows Git Bash, prefix with `MSYS_NO_PATHCONV=1` and use `docker run --rm` (no `-it`) if the terminal is non-interactive.

## Testing all functions in a project

Run the one-shot build from each function directory. Example for a project with `showcase_app` and functions `health`, `get_items`, `save_item`, `get_file`, `put_file`, `event_handler`:

```bash
CODE=path/to/code/showcase_app/functions
for fn in health get_items save_item get_file put_file event_handler; do
  (cd "$CODE/$fn" && MSYS_NO_PATHCONV=1 docker run --rm \
    -v "$(pwd)/out:/out" \
    --mount type=bind,src="$(pwd)",dst=/src_ro,ro \
    --mount type=tmpfs,dst=/src \
    taubyte/go-wasi /bin/bash -c "rsync -a --delete /src_ro/ /src/ 2>/dev/null || cp -a /src_ro/. /src/; cd /src && source /utils/wasm.sh && export FILENAME=$fn && build \"\${FILENAME}\"; echo BUILD_EXIT=\$?")
done
```

On Windows, use the same loop in Git Bash with `MSYS_NO_PATHCONV=1` before `docker run`.

## Requirements

- Docker installed and running.
- Function directory contains `.taubyte/config.yaml`, `.taubyte/build.sh`, `go.mod`, and the function `.go` file(s).
- Path layout: `<app_name>/functions/<function_name>/` at code repo root (see `taubyte-folder-examples.md`).

## TinyGo optimizer crash (SIGSEGV)

If remote or Docker build fails with **"fatal error: unexpected signal during runtime execution [signal SIGSEGV]"** in `tinygo.org/x/go-llvm` (e.g. `LLVMRunFunctionPassManager` or `LLVMVerifyModule`) during the optimize step, the TinyGo/LLVM optimizer can crash on some code patterns. **Mitigation:** Simplify the function to avoid heavy use of `map[string]interface{}`, `interface{}` slices, or complex nested maps; use **concrete structs** with JSON tags and `[]Struct` instead of `[]map[string]string` or `map[string]interface{}`. If it still crashes, consider reporting to Taubyte/TinyGo with a minimal repro.

## See also

- `check-logs-debug.md` — How to check logs and debug (tau query builds, tau query logs &lt;id&gt;).
- `taubyte-folder-examples.md` — Code repo layout and .taubyte structure.
- `sdk-api-must.md` — Correct go-sdk usage to fix compile errors.
