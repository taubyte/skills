---
name: configuring-taubyte-build-runtime
description: Configures the per-resource build/runtime files Taubyte runs server-side — `.taubyte/config.yaml` (image + workflow + similar metadata) and `.taubyte/build.sh` (the actual build script, where build-time environment variables also live as `export …`). Covers Go function builds (`taubyte/go-wasi:latest|v2`), website builds tailored to the actual stack (Vite `dist/`, CRA `build/`, plain static), and library builds. Use when adding/editing build scripts, declaring env vars, switching build images, or debugging "build succeeds but produces nothing" / "env var is empty in handler".
---

# Configuring Taubyte Build & Runtime

## When to use

- Adding a `.taubyte/build.sh` to a function/website/library repo for the first time
- Switching the build image (e.g. `taubyte/go-wasi:latest` → `taubyte/go-wasi:v2`)
- Declaring environment variables that the build needs to see
- A website builds but serves nothing (empty `/out`)
- A function compiles to WASM but the runtime sees an empty / wrong env value
- The build script for one stack (Vite) was copy-pasted onto another (CRA) and now the bundle output isn't found

## The two files (every Taubyte buildable resource has them)

```text
<resource>/
└── .taubyte/
    ├── config.yaml      # image + workflow metadata (NOT env vars)
    └── build.sh         # the actual build script (where env vars live)
```

Where `<resource>` is one of:

| Resource | Path |
| --- | --- |
| Function | `<project>/code/.../functions/<name>/.taubyte/` |
| Website | `<project>/websites/<repo>/.taubyte/` |
| Library | `<project>/libraries/<repo>/.taubyte/` |

## Hard rules

1. **`.taubyte/config.yaml`** is for **build metadata** (image, workflow). It is **not** a place to declare runtime env vars.
2. **`.taubyte/build.sh`** is the **only** correct place for build-time env vars. Use `export NAME=value` at the top, before the build commands.
3. **`.taubyte/build.sh` must be non-empty and executable.** An empty `build.sh` "builds successfully" while producing nothing — this is the most common silent failure.
4. **Websites must write deployable static output to `/out`.** See [building-taubyte-websites](../building-taubyte-websites/SKILL.md).
5. **Don't guess the website framework's output directory.** Vite emits `dist/`, Create React App emits `build/`, Next.js emits `.next/` (then exports), etc. Read `package.json` (`scripts.build`) to confirm before scripting.

## Function — `config.yaml` (canonical pattern)

```yaml
version: 1.00
environment:
  image: taubyte/go-wasi:latest
workflow:
  - build
```

Picking an image:

| Image | Use for |
| --- | --- |
| `taubyte/go-wasi:latest` | Default for Go serverless functions and libraries (TinyGo + WASI; handlers use `package lib` in the usual scaffold). |
| `taubyte/go-wasi:v2` | Pinned major when you need stability across CI runs (image/layout may differ from `latest`; only switch when you intend to). |

If you change the image here, **also use the same image** for any local Docker WASM verify (see [verifying-taubyte-functions](../verifying-taubyte-functions/SKILL.md)) so local and cloud builds match.

## Function — `build.sh` (canonical pattern)

```bash
#!/bin/bash

# taubyte/go-wasi:latest: non-login build environments (Dream/monkey, some Docker invocations) often lack
# go/tinygo on default PATH — export before wasm.sh or builds fail ("go: command not found", tinygo missing).
export PATH="/usr/local/go/bin:/usr/local/tinygo/bin:${PATH}"

. /utils/wasm.sh

# Build-time env vars for this resource — declare here, NOT in config.yaml:
# export API_BASE_URL="https://api.example.com"
# export FEATURE_FLAG_X=1

build "${FILENAME}"
ret=$?
echo -n $ret > /out/ret-code
exit $ret
```

What this does:

- `export PATH=...` makes **`taubyte/go-wasi:latest`** reliably find **`go`** and **`tinygo`** when the shell is not a full login environment (matches what [verifying-taubyte-functions](../verifying-taubyte-functions/SKILL.md) does inside Docker verify).
- `. /utils/wasm.sh` sources the toolchain helpers provided by the `taubyte/go-wasi` image.
- `build "${FILENAME}"` invokes the helper that compiles the function into WASM. **`FILENAME`** must name the **actual entry `.go` file** at the function root (the CLI scaffold uses **`empty.go`**). Renaming the file without updating **`FILENAME`** (or renaming `empty.go` → `lib.go` hoping to fix `package lib` noise) often yields **TinyGo / verify errors** or **missing `artifact.wasm`** on remote builds — **stay on `empty.go`** unless you have an end-to-end verified change for both the file name and `build.sh`.
- The script writes the build's exit code to `/out/ret-code` and exits with the same code (Taubyte uses both signals).

## Library — `build.sh` and `config.yaml`

Libraries follow the **same shape as functions** (typically Go + `taubyte/go-wasi:latest`). Most generated library scaffolds work as-is. Edit only when you need extra build env or a different image.

## Website — `config.yaml` (typical)

Websites usually use a Node-based image. Example minimal `config.yaml` (the exact image string may vary by template):

```yaml
version: 1.00
environment:
  image: node:20-alpine
workflow:
  - build
```

If the template generated a different image string, leave it as-is unless you have a reason to change.

## Website — `build.sh` patterns by stack

The script must end with deployable static files in **`/out`**. Read `package.json`'s `scripts.build` to find the real output directory before scripting. Do not assume.

### Plain static HTML (no bundler)

```bash
#!/bin/bash
cp index.html /out/
exit $?
```

### Vite (default output `dist/`)

```bash
#!/bin/bash
set -euo pipefail
npm ci
npm run build
cp -r dist/. /out/
```

### Create React App / `react-scripts` (output `build/`)

```bash
#!/bin/bash
set -euo pipefail
npm ci
npm run build
cp -r build/. /out/
```

### Other stacks

For Next, Nuxt, custom webpack, etc. — run that stack's install + production build commands, then `cp -r <framework-output-dir>/. /out/` using the directory **that tool** documents. If unsure, ask rather than guess.

### Same-origin website + API pattern (Vite + HTTPS functions)

When the website and the HTTPS functions share a domain (`/api/...` routes on the same configured domain):

- In `vite.config.ts`, set `envPrefix: ["VITE_", "APP_"]` if the client reads `APP_*` build-time vars.
- Default the API base to `window.location.origin` so the browser hits same-origin functions after deploy.
- Document `APP_API_BASE_URL` (or whatever name you pick) in `.env.example` for Dream/local where the API may be on another host or port.
- In project config, attach the website and the HTTPS functions to the **same `domains`** entry so relative `/api/...` calls resolve.

## Build-time env vars (the only correct place is `build.sh`)

```bash
#!/bin/bash
set -euo pipefail

# Declare env vars HERE, before the build commands run:
export VITE_PUBLIC_BASE="/"
export APP_API_BASE_URL="${APP_API_BASE_URL:-}"
export NODE_OPTIONS="--max-old-space-size=4096"

npm ci
npm run build
cp -r dist/. /out/
```

Things to know:

- These vars are **build-time** (visible to the bundler / compiler), not runtime container env in the running cloud.
- The bundler will inline values that match its prefix (e.g. Vite inlines `VITE_*` and any prefixes you declare in `envPrefix`).
- Secrets baked at build time end up in the deployed bundle. Don't use `build.sh` env for secrets that must stay private — surface those through the platform's secret mechanism instead.

## Common build outputs (where things land)

| Resource | What `build.sh` should produce |
| --- | --- |
| Function | Compiled WASM written by the `build` helper; `ret-code` in `/out` |
| Library | Compiled WASM (same shape as a function); `ret-code` in `/out` |
| Website | Whatever your stack emits, copied into `/out` (HTML/CSS/JS/assets) |

## Validating before push

```
Pre-push validation progress:
- [ ] .taubyte/config.yaml exists and parses as YAML
- [ ] .taubyte/build.sh exists, is non-empty, executable (chmod +x)
- [ ] build.sh references the real output dir of this stack (Vite -> dist/, CRA -> build/, ...)
- [ ] Env vars (if any) are exported in build.sh, NOT declared in config.yaml
- [ ] For functions: image in config.yaml matches the local verify image (if local verify is in scope)
- [ ] git push the resource repo
- [ ] Remote: wait for webhook build / Dream: dream inject push-specific
```

## Recovery — symptom → likely cause

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| Remote function build: `.taubyte` not found under `.../<app>/functions/<name>` | Sources live under **`code/applications/<app>/functions/...`** but the builder expects **`code/<app>/functions/...`** (or the path in the log differs) | Move the function folder to the path shape in the error; see [creating-taubyte-resources](../creating-taubyte-resources/SKILL.md) |
| Function build: TinyGo errors / no `artifact.wasm` after renaming `empty.go` | **`FILENAME`** / entry file / image expectations out of sync | Revert to scaffold **`empty.go`** and `build "${FILENAME}"` with default `FILENAME`; keep `export PATH=...` for go/tinygo |
| Website "deploys" but serves blank / 404 | `/out` is empty after build.sh | Confirm the framework output dir; copy from there into `/out` |
| Bundler env value is empty in browser | Var declared in `config.yaml` instead of `build.sh` | Move to `export NAME=value` in `build.sh` |
| Function builds but old behavior persists | Not pushed / not injected on Dream | `git push` then `dream inject push-specific` (see [triggering-dream-builds](../triggering-dream-builds/SKILL.md)) |
| Cloud build uses wrong toolchain version | `image:` in `config.yaml` not updated | Edit `config.yaml`; also update local verify image to match |
| Env var fine in CI but missing in deployed bundle | Bundler env-prefix policy excludes it (e.g. Vite without `envPrefix`) | Add the prefix to bundler config or rename the var |
| `build.sh` runs but exits non-zero silently | Missing `set -euo pipefail` masks earlier failures | Add it at the top of every Node-based `build.sh` |

## Gotchas

- **Env vars in `config.yaml` do nothing for build-time use.** Always `export` them in `build.sh`.
- **Vite-only `cp -r dist/* /out/` on a CRA repo silently produces an empty `/out`.** Always confirm `package.json`'s `scripts.build` output directory.
- **`cp -r dist /out/` puts files under `/out/dist/`** instead of into `/out/`. Use `cp -r dist/. /out/` (or `cp -a dist/. /out/`) to copy the **contents** of the directory.
- **Empty `build.sh`** is the silent killer for `--template empty` resources. Always populate before push.
- **Forgetting `chmod +x build.sh`** can cause "permission denied" on some build images. Templates usually mark it executable; new files you create may need it.
- **Local verify image drift.** If `config.yaml` says `:v2` but you run the Docker WASM recipe with `:latest`, you may not reproduce a cloud failure locally.
- **Editing `build.sh` beside `index.html`.** A mistaken paste can merge **HTML into the shell script** — especially on websites. The script must stay valid bash only; see [building-taubyte-websites](../building-taubyte-websites/SKILL.md).

## Related skills

- `building-taubyte-websites` — the `/out` rule + minimal HTML baseline
- `verifying-taubyte-functions` — local Docker WASM verify with the matching image
- `writing-taubyte-functions` — the Go SDK + handler authoring this script builds
- `creating-taubyte-resources` — `tau new` flag patterns for the resources whose builds this skill configures
- `triggering-dream-builds` — push the rebuilt resource into Dream after editing
- `pushing-taubyte-projects` — push the repo containing the edited `.taubyte/`
