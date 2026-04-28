---
name: tb_sdk_go_local_wasm_docker
description: Local Go WASM verify via taubyte/go-wasi:v2 Docker recipe (not tau build).
---

## Policy

- Do **not** use `tau build` / `tau run` unless the user explicitly asked.
- Fix issues by editing **root `empty.go`**, `go.mod`, and `.taubyte/*` only.

## Docker recipe

From the **function root** (`empty.go` + `go.mod`):

```bash
mkdir -p out
docker run -it --rm \
  -e CODE=/src \
  -v "$(pwd)/out:/out" \
  --mount type=bind,src="$(pwd)",dst=/src_ro,ro \
  --mount type=tmpfs,dst=/src \
  taubyte/go-wasi:v2 /bin/bash -c '
    set -e
    rsync -a --delete /src_ro/ /src/ 2>/dev/null || cp -a /src_ro/. /src/
    echo "export CODE=/src; cd /src; source /utils/wasm.sh" > /tmp/cowrc
    exec bash --rcfile /tmp/cowrc -i'
```

- `CODE=/src` is required for `/utils/wasm.sh`.
- Windows Git Bash: `MSYS_NO_PATHCONV=1` or `MSYS2_ARG_CONV_EXCL='*'` before `docker`; use `$(pwd -W)` for bind mounts if Docker Desktop needs Windows paths.
- If module download fails in non-interactive runs, set:
  ```bash
  GOSUMDB=sum.golang.org
  GOPROXY=https://proxy.golang.org,direct
  ```

See `tb_sys_core_constraints` for image pin **taubyte/go-wasi:v2**.