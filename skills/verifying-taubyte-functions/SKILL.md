---
name: verifying-taubyte-functions
description: Verifies a Taubyte Go function locally via the `taubyte/go-wasi` Docker recipe (preferred over `tau build`, with tmpfs+bind-mount-ro to avoid root-owned artifacts in the source tree), and verifies a function actually serves on Dream by curling the gateway with the right `Host:` header (plus `/etc/hosts` mapping for `*.localtau`). Use when locally compiling a Go function to WASM, when smoke-testing a function before pushing, or when probing a Dream-hosted HTTP function from the laptop.
---

# Verifying Taubyte Functions

## When to use

- "Does this Go function compile to WASM?"
- "Smoke-test the function before pushing"
- "Check whether the deployed function actually responds"
- Need to hit a `*.localtau` URL from the browser
- Avoiding root-owned artifacts left behind by `tau build`

## Two distinct things

| Goal | Use this section |
| --- | --- |
| Compile WASM **locally** to confirm the source builds | "Local Go WASM verify (Docker)" below |
| Hit a function **running on Dream** | "Runtime verification (Dream gateway)" below |

`tau build` exists, but its Docker workflow tends to leave root-owned artifacts (`go.sum`, `lib/`, `main.go`) in the function tree. The Docker recipe below avoids that.

## Local Go WASM verify (Docker)

Run from the function root (the directory containing `go.mod` and the source `.go` files):

```bash
cd <project>/code/functions/<fn>

mkdir -p out

docker run --rm \
  -e CODE=/src \
  -e GOPROXY=https://proxy.golang.org,direct \
  -e GOSUMDB=sum.golang.org \
  -v "$(pwd)/out:/out" \
  --mount type=bind,src="$(pwd)",dst=/src_ro,ro \
  --mount type=tmpfs,dst=/src \
  taubyte/go-wasi:latest /bin/bash -lc '
    set -euo pipefail
    rsync -a --delete /src_ro/ /src/ 2>/dev/null || cp -a /src_ro/. /src/
    cd /src
    export CODE=/src
    # go-wasi:latest: go/tinygo live under /usr/local/* but are often missing from PATH in non-login shells (same class of env Dream uses when running build.sh).
    export PATH="/usr/local/go/bin:/usr/local/tinygo/bin:${PATH}"
    source /utils/wasm.sh
    ls -la /out
  '
```

Why this shape:

- `--mount type=bind,...,ro` makes your real source read-only inside the container.
- `--mount type=tmpfs,dst=/src` provides a writable scratch copy at `/src`.
- The script `rsync`s the read-only source into the tmpfs, builds there, and writes only into the host-mapped `out/` directory.
- Result: **no root-owned files** appear in your real source tree.

### Notes

- The function's `.taubyte/config.yaml` declares the build image (e.g. `taubyte/go-wasi:latest` or `taubyte/go-wasi:v2`). **Use the same image** in the local recipe as the cloud will use.
- Don't use `-it` in non-TTY environments — it fails with `cannot attach stdin to a TTY-enabled container`. The recipe above already omits `-it`.
- The recipe is intended only when the user explicitly asks for local build verification. Default Taubyte workflow is "push to GitHub → cloud builds".
- **`taubyte/go-wasi:latest` + `PATH`:** the image installs **`go`** and **`tinygo`** under `/usr/local/go/bin` and `/usr/local/tinygo/bin`. Non-login shells (this Docker block, Dream/monkey running `.taubyte/build.sh`) often start with a **minimal `PATH`**, so `/utils/wasm.sh` can fail with **`go: command not found`** unless you export `PATH` (shown in the recipe above) or put the same `export` at the top of **`.taubyte/build.sh`** before `. /utils/wasm.sh` — see [configuring-taubyte-build-runtime](../configuring-taubyte-build-runtime/SKILL.md).

### Output expectation

After the run, `out/` on the host should contain at least the WASM artifact and any expected build outputs. If `out/` is empty, the build failed; re-read the container's stdout for compile errors.

## Runtime verification (Dream gateway)

To curl a function running on Dream, you need (1) the gateway port and (2) the function's domain FQDN.

```bash
# 1. Discover gateway port
# `dream status gateway` often prints a blank line before `@ http://...`; awk '/@ http/{print $3}' can return empty — prefer:
GW=$(dream status gateway default | grep -Eo 'http://[0-9.]+:[0-9]+' | head -n1)

# 2. Read the function's FQDN from its domain YAML
FQDN=$(awk '/^fqdn:/{print $2; exit}' config/domains/<domain>.yaml)

# 3. Curl with Host header
curl -i -H "Host: $FQDN" "$GW/<path>"
```

Concrete example for a `ping_pong`-templated function on `/ping`:

```bash
curl -i -H "Host: <random>.default.localtau" "http://127.0.0.1:<gateway_port>/ping"
# Expected: HTTP 200 and body "PONG"
```

## Hosts file for `*.localtau` (browser convenience)

Browsers don't let you set a `Host:` header trivially. To hit `http://<fqdn>/<path>` directly, map the generated FQDN to localhost:

```bash
sudo sh -c 'echo "127.0.0.1 <random>.default.localtau" >> /etc/hosts'
# or edit /etc/hosts manually:
# 127.0.0.1 <random>.default.localtau
```

After the mapping, browsing to `http://<random>.default.localtau:<gateway_port>/<path>` works directly.

Re-run for each new generated FQDN you want to open in a browser. To find the FQDN:

```bash
awk '/^fqdn:/{print $2; exit}' config/domains/<domain>.yaml
```

## Workflow checklists

### Pre-push local smoke

```
Smoke check progress:
- [ ] cd into function root (has go.mod)
- [ ] Run Docker WASM build recipe (above)
- [ ] Confirm out/ contains expected artifacts
- [ ] Inspect container stdout for errors
- [ ] If green, proceed to push
```

### Post-deploy runtime check

```
Runtime check progress:
- [ ] dream status gateway <universe> -> note port (use grep -Eo 'http://[0-9.]+:[0-9]+' if awk returns empty)
- [ ] Read FQDN from config/domains/<domain>.yaml
- [ ] curl -i -H "Host: <fqdn>" http://127.0.0.1:<port>/<path>
- [ ] Expect HTTP 200 + expected body
- [ ] If non-200, see diagnosing-dream-builds for build/log inspection
```

## Gotchas

- **Don't run `tau build` casually.** It works, but it can leave **root-owned** artifacts (`lib/`, `main.go`, `artifact.zip`) in the function tree. **Never commit those** — they break Dream/go-wasi builds (mixed packages, import cycles). Prefer the Docker recipe above; delete + `git rm --cached` if they appear, and add `.gitignore` patterns for them.
- **Wrong `Host:` header = 404.** Dream's gateway routes by `Host:`; without the right header you'll see `404` even with a healthy function. Re-read the FQDN from the domain YAML.
- **Stale gateway port across sessions.** `dream status gateway` is the canonical source; ports change between Dream runs.
- **Generated FQDNs only resolve via hosts mapping.** They aren't real DNS; the browser must be told `127.0.0.1`.
- **Remote vs Dream**: this skill is Dream-focused. For remote clouds, hit the FQDN directly over the public DNS the cloud configured.

## Related skills

- `inspecting-dream-status` — gateway port discovery
- `triggering-dream-builds` — push and inject before runtime checks
- `diagnosing-dream-builds` — when a runtime check fails
- `building-taubyte-websites` — the matching `Host:`-header pattern for sites
