# Taubyte .taubyte Folder Examples

This document provides complete examples of `.taubyte/` folder contents for each resource type in Taubyte.

## Code Repository Layout (with Application) — CRITICAL

When the project uses an **application** and functions are under that app, the **build pipeline** expects function code at the **root** of the code repo as:

```
<app_name>/
└── functions/
    └── <function_name>/
        ├── .taubyte/
        │   ├── config.yaml
        │   └── build.sh
        ├── go.mod
        └── <function_name>.go
```

- **Correct:** `showcase_app/functions/health/`, `Backend/functions/get_leaderboard/` (at repo root).
- **Wrong:** `applications/showcase_app/functions/health/` — the builder does **not** look under `applications/`. Using that layout causes: _"no taubyte directory found in .../showcase_app/functions/event_handler"_.

If `tau new function` creates under `applications/<app_name>/functions/<name>/`, move the content to `<app_name>/functions/<name>/` at the code repo root and remove the empty `applications/` directory so builds succeed.

## Build / Test Functions Locally (Docker)

To catch compile and SDK errors before pushing, run the same build image as CI from each function directory. **Full steps (Linux/macOS/Windows), interactive shell, and testing all functions:** see **`build-test-functions-docker.md`**.

Quick one-shot (replace `<function_name>`; on Windows prefix with `MSYS_NO_PATHCONV=1`):

```bash
cd code/<app_name>/functions/<function_name>
docker run --rm -v "$(pwd)/out:/out" --mount type=bind,src="$(pwd)",dst=/src_ro,ro --mount type=tmpfs,dst=/src \
  taubyte/go-wasi /bin/bash -c 'rsync -a /src_ro/ /src/ 2>/dev/null || cp -a /src_ro/. /src/; cd /src && source /utils/wasm.sh && export FILENAME=<function_name> && build "${FILENAME}"; echo BUILD_EXIT=$?'
```

Success: `BUILD_EXIT=0`.

## Overview

All resources that require build configuration use a `.taubyte/` folder containing:

- `config.yaml` - Build environment and workflow configuration
- `build.sh` - Build script that outputs to `/out` directory

**CRITICAL**: All build outputs MUST go to `/out` directory.

---

## HTTP Function

**Location**: `functions/function_name/.taubyte/`

### `.taubyte/config.yaml`

```yaml
version: 1.00
environment:
  image: taubyte/go-wasi:latest
  variables:
workflow:
  - build
```

### `.taubyte/build.sh`

```bash
#!/bin/bash

. /utils/wasm.sh

build "${FILENAME}"
ret=$?
echo -n $ret > /out/ret-code
exit $ret
```

**Notes:**

- The `FILENAME` variable is automatically set by Taubyte to match the function name
- Build output goes to `/out` directory
- Uses `taubyte/go-wasi:latest` image for WebAssembly compilation

### Directory Structure

```
functions/
└── my_http_function/
    ├── .taubyte/
    │   ├── config.yaml
    │   └── build.sh
    ├── go.mod
    ├── go.sum
    └── my_http_function.go
```

---

## PubSub Function

**Location**: `functions/function_name/.taubyte/`

### `.taubyte/config.yaml`

```yaml
version: 1.00
environment:
  image: taubyte/go-wasi:latest
  variables:
workflow:
  - build
```

### `.taubyte/build.sh`

```bash
#!/bin/bash

. /utils/wasm.sh

build "${FILENAME}"
ret=$?
echo -n $ret > /out/ret-code
exit $ret
```

**Notes:**

- Same structure as HTTP functions
- The `FILENAME` variable is automatically set by Taubyte to match the function name
- Build output goes to `/out` directory

### Directory Structure

```
functions/
└── pubsub_handler/
    ├── .taubyte/
    │   ├── config.yaml
    │   └── build.sh
    ├── go.mod
    ├── go.sum
    └── pubsub_handler.go
```

---

## P2P Function

**Location**: `functions/function_name/.taubyte/`

### `.taubyte/config.yaml`

```yaml
version: 1.00
environment:
  image: taubyte/go-wasi:latest
  variables:
workflow:
  - build
```

### `.taubyte/build.sh`

```bash
#!/bin/bash

. /utils/wasm.sh

build "${FILENAME}"
ret=$?
echo -n $ret > /out/ret-code
exit $ret
```

**Notes:**

- Same structure as HTTP and PubSub functions
- The `FILENAME` variable is automatically set by Taubyte to match the function name
- Build output goes to `/out` directory

### Directory Structure

```
functions/
└── p2p_handler/
    ├── .taubyte/
    │   ├── config.yaml
    │   └── build.sh
    ├── go.mod
    ├── go.sum
    └── p2p_handler.go
```

---

## Library

**Location**: `tb_library_name/.taubyte/` (in library repository root)

### `.taubyte/config.yaml`

```yaml
version: 1.00
enviroment:
  image: golang:1.21-alpine
  variables:
workflow:
  - build
```

**Note**: The documentation uses `enviroment` (typo) instead of `environment`. Use `enviroment` to match existing documentation.

### `.taubyte/build.sh`

```bash
#!/bin/bash
go mod download
go build -o /out/library.a .
```

**Notes:**

- Build output must go to `/out` directory
- Uses `golang:1.21-alpine` image
- Outputs a library archive file

### Directory Structure

```
tb_library_mylibrary/
├── .taubyte/
│   ├── config.yaml
│   └── build.sh
├── go.mod
└── shared_code.go
```

---

## Website

**Location**: `tb_website_name/.taubyte/` (in website repository root)

### `.taubyte/config.yaml`

```yaml
version: 1.00
enviroment:
  image: node:16.18.0-alpine
  variables:
workflow:
  - build
```

**Note**: The documentation uses `enviroment` (typo) instead of `environment`. Use `enviroment` to match existing documentation.

### `.taubyte/build.sh`

**CRITICAL:** NEVER leave website `build.sh` empty or without copying to `/out`. The pipeline serves only what is in `/out`; an empty script means nothing is deployed.

**Static website (no package.json / no build step):** Copy files into `/out`:

```bash
#!/bin/bash
# Static site: copy content to /out (required by Taubyte)
cp index.html /out/
[ -e favicon.ico ] && cp favicon.ico /out/
[ -e README.md ] && cp README.md /out/
```

**Node/Vue/React (has package.json, build step):**

```bash
#!/bin/bash
yarn install
yarn audit fix
yarn build && mv dist/* /out
```

**Alternative with npm:**

```bash
#!/bin/bash
npm install
npm audit fix
npm run build && cp -r dist/* /out
```

**Notes:**

- Build output MUST go to `/out` directory
- Uses `node:16.18.0-alpine` image (adjust version as needed)
- Must move/copy build artifacts to `/out` directory

### Directory Structure

```
tb_website_mywebsite/
├── .taubyte/
│   ├── config.yaml
│   └── build.sh
├── package.json
├── vue.config.js          # or react config, etc.
├── public/
│   ├── favicon.ico
│   └── index.html
└── src/
    ├── App.vue            # or App.jsx, etc.
    ├── main.js
    └── components/
```

---

## Key Points

### All Resources

1. **Build Output**: All build scripts MUST output to `/out` directory
2. **Config Location**: `.taubyte/` folder goes in the resource root directory
3. **Build Script**: Must be executable and output to `/out`

### Functions (HTTP, PubSub, P2P)

- All use the same `.taubyte/` structure
- Use `taubyte/go-wasi:latest` image
- Use `/utils/wasm.sh` helper script
- `FILENAME` variable is automatically set by Taubyte

### Libraries

- Use `golang:1.21-alpine` image (or appropriate Go version)
- Build Go library archive
- Output to `/out/library.a` or similar

### Websites

- Use Node.js Docker image (e.g., `node:16.18.0-alpine`)
- **build.sh must never be empty:** it must copy or build output into `/out` or the site will not deploy
- Static site (only index.html, etc.): `cp index.html /out/` (and any other static files)
- Node/Vue/React: install dependencies, build, then copy `dist/*` to `/out`
- Can use `yarn` or `npm` for built sites

---

## Common Build Script Patterns

### Function Build Script (Standard)

```bash
#!/bin/bash

. /utils/wasm.sh

build "${FILENAME}"
ret=$?
echo -n $ret > /out/ret-code
exit $ret
```

### Library Build Script (Go)

```bash
#!/bin/bash
go mod download
go build -o /out/library.a .
```

### Website Build Script (Yarn)

```bash
#!/bin/bash
yarn install
yarn audit fix
yarn build && mv dist/* /out
```

### Website Build Script (npm)

```bash
#!/bin/bash
npm install
npm audit fix
npm run build && cp -r dist/* /out
```

### Website Build Script (Static – no npm/yarn)

For a site that is only static files (e.g. `index.html`), **do not leave build.sh empty**. Copy to `/out`:

```bash
#!/bin/bash
# Static site: copy content to /out (required by Taubyte)
cp index.html /out/
[ -e favicon.ico ] && cp favicon.ico /out/
[ -e README.md ] && cp README.md /out/
```

---

## Validation Checklist

Before deploying, verify:

- [ ] `.taubyte/config.yaml` exists and is valid YAML
- [ ] `.taubyte/build.sh` exists and is executable
- [ ] Build script outputs to `/out` directory
- [ ] Docker image in `config.yaml` matches project requirements
- [ ] For functions: `FILENAME` variable is used correctly
- [ ] For websites: **build.sh is not empty**; build artifacts are copied to `/out` (static: `cp index.html /out/`; Node: `npm run build && cp -r dist/* /out`)
- [ ] For libraries: Build output is placed in `/out`

---

## Troubleshooting

### Build Fails

1. Check that build script outputs to `/out`
2. Verify Docker image version matches project needs
3. Ensure all dependencies are properly specified
4. Check build logs for specific error messages

### Output Not Found

- Ensure build script explicitly copies/moves output to `/out`
- Verify output directory path in build script
- Check that build command actually produces output

### Website build produces nothing / site not loading

- **Cause:** Website `.taubyte/build.sh` may be empty or not copying to `/out`. The pipeline only serves content from `/out`.
- **Fix:** Edit `tb_website_<name>/.taubyte/build.sh`. For static sites (only index.html): add `cp index.html /out/` (and optional `[ -e favicon.ico ] && cp favicon.ico /out/`). For Node/Vue/React: use `npm run build && cp -r dist/* /out` (or yarn equivalent). See "Website" section above and "Website Build Script (Static)" in Common Build Script Patterns.

### Function Build Issues

- Verify `FILENAME` variable is used (automatically set by Taubyte)
- Check that `/utils/wasm.sh` is sourced correctly
- Ensure Go code is properly formatted for WebAssembly
