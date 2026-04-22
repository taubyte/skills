---
name: taubyte-build-runtime-config
description: Manages server-side build/runtime config through `.taubyte/build.sh` and `.taubyte/config.yaml`, including env vars when needed.
---

# Build Runtime Config

## When to Use

Use whenever a resource needs server-side build logic, runtime variables, or deployment-time configuration.
For Go functions, also load `taubyte-go-sdk-constraints`.

## Targets

- Function: `<project>/code/.../functions/<name>/.taubyte/`
- Website repo: `<project>/websites/tb_website_<name>/.taubyte/`
- Library repo: `<project>/libraries/tb_library_<name>/.taubyte/`

## Required checks

1. `.taubyte/config.yaml` exists and is valid.
2. `.taubyte/build.sh` exists, is non-empty, and writes output expected by Taubyte.
3. Runtime env vars (if needed) are declared in config and used by code/build.

## Function config pattern

```yaml
version: 1.00
environment:
  image: taubyte/go-wasi:latest
  variables:
    API_BASE_URL: "https://example.com"
workflow:
  - build
```

## Function build.sh pattern

```bash
#!/bin/bash
. /utils/wasm.sh
build "${FILENAME}"
ret=$?
echo -n $ret > /out/ret-code
exit $ret
```

## Website build.sh pattern

- Must write final assets to `/out` (never leave empty).
- Example static:
  ```bash
  #!/bin/bash
  cp index.html /out/
  ```
- Example Node/Vite:
  ```bash
  #!/bin/bash
  npm ci
  npm run build
  cp -r dist/* /out/
  ```

## Rules

- If env vars are required by server-side behavior, add them in `.taubyte/config.yaml` and document in context log.
- Do not push code before validating `.taubyte/build.sh` and `.taubyte/config.yaml`.
- Re-run build/log verification after config changes.
- For Go compile issues, validate handler code against `taubyte-go-sdk-constraints`.
