---
name: tb_sys_build_runtime_config
description: Manages server-side build/runtime via `.taubyte/build.sh` and `.taubyte/config.yaml`; env vars live in build.sh only. Website build.sh is stack-specific (Vite vs CRA/React, etc.). Documents GitHub → webhook vs Dream inject (push-specific).
---

# Build runtime config

## When to use

Use whenever a resource needs **server-side build logic**, **runtime environment variables**, or **deployment-time** setup. For Go functions, also load **`tb_sdk_go_*`**.

---

## How builds are triggered

1. **Resource code and config** are pushed to **GitHub** (normal `git push` workflow for the function, website, or library repo as appropriate).

2. **Remote cloud**  
   After the push, the cloud is reached from GitHub (webhook). **Builds trigger automatically** — no extra inject step on your machine.

3. **Local Dream**  
   GitHub **cannot** call into your laptop, so **webhooks do not drive Dream**. After pushing to GitHub, trigger the build on the local Dream universe using **Dream inject**:

   | Command | Use for |
   |---------|--------|
   | **`dream inject push-specific`** | Trigger a build for an individual **resource repo** (website/library) in Dream after the code is pushed to GitHub. |

   **Default policy:** do **not** run local builds/tests. The only local action is **`dream inject push-specific`** (Dream build trigger) unless the user explicitly requests local build/verification.

---

## Targets

- Function: `<project>/code/.../functions/<name>/.taubyte/`
- Website repo: `<project>/websites/tb_website_<name>/.taubyte/`
- Library repo: `<project>/libraries/tb_library_<name>/.taubyte/`

---

## Required checks

1. **`.taubyte/config.yaml`** exists and is valid (image, workflow, etc. — **not** runtime env var declarations).
2. **`.taubyte/build.sh`** exists, is non-empty, writes output Taubyte expects (e.g. function wasm + ret-code, website assets under `/out`).
3. **Runtime env vars** (if any) are set in **`.taubyte/build.sh`** (e.g. `export …` before build steps), **not** in `config.yaml`.

---

## Function `config.yaml` pattern

```yaml
version: 1.00
environment:
  image: taubyte/go-wasi:latest
workflow:
  - build
```

Keep **`config.yaml`** for image/workflow and similar metadata only.

---

## Function `build.sh` pattern

```bash
#!/bin/bash
. /utils/wasm.sh
# Runtime env for this build (example — use real names/values your code expects):
# export MY_VAR=value
build "${FILENAME}"
ret=$?
echo -n $ret > /out/ret-code
exit $ret
```

---

## Website `build.sh` (technology-specific)

Website **`build.sh` is not one generic script** — it must match **how that repo is built** (package manager, `package.json` scripts, and especially **where the bundler writes static files**). Treating every Node site like **Vite** (always `dist/`) is wrong for **Create React App** and other stacks that emit **`build/`** or another directory.

**Always**

- Produce a **non-empty** **`/out`** (Taubyte’s deploy staging path inside the build environment — not the same as a framework’s own `out` folder name on disk).
- Put **build-time env** here with **`export`**, not in `config.yaml`.

**Pick the pattern from the repo**

Read **`package.json`** (`scripts.build` or equivalent) and the framework docs to learn the **real output directory**, then copy **that** tree into `/out`.

**Examples (illustrative — adjust to the project)**

- **Plain static** (no bundler):

  ```bash
  #!/bin/bash
  cp index.html /out/
  ```

- **Vite** (default output **`dist/`**):

  ```bash
  #!/bin/bash
  npm ci
  npm run build
  cp -r dist/* /out/
  ```

- **Vite + same-origin API** (static site and **`/api/...`** handlers on the **same** configured domain — typical full-stack Taubyte):

  - In **`vite.config.ts`**, use **`envPrefix: ["VITE_", "APP_"]`** when the client reads **`APP_*`** at build time (e.g. optional API base override).
  - Prefer **`window.location.origin`** as the default API base when unset so the browser hits **same-origin** HTTP functions after deploy.
  - Document **`APP_API_BASE_URL`** (or equivalent) in **`.env.example`** for Dream/local when the API is on another host or port.
  - In **config**, attach **website** and **HTTPS functions** to the **same `domains`** entry so relative **`/api/...`** calls resolve.

- **Create React App / `react-scripts`** (output **`build/`**, not `dist/`):

  ```bash
  #!/bin/bash
  npm ci
  npm run build
  cp -r build/* /out/
  ```

- **Other stacks** (Next, Nuxt, custom webpack, etc.): run that project’s install + production build commands, then **`cp -r <framework-output-dir>/* /out/`** using the directory **that tool** documents. If you are unsure, **do not guess** — ask the user for the repo’s intended build output directory or rely on existing project docs/config.

---

## Rules

- **Env vars:** declare and use them in **`.taubyte/build.sh`** only; do **not** put runtime env declarations in **`.taubyte/config.yaml`** for this purpose.
- **Websites:** tailor **`build.sh`** to the **actual** framework and build output (e.g. **Vite → `dist/`**, **CRA → `build/`**); never copy a Vite-only script onto a React-CRA repo without changing paths and commands.
- Do not push to GitHub before validating **`.taubyte/build.sh`** and **`.taubyte/config.yaml`**.
- After changing build/config, **always push to GitHub**. Then:
  - **Remote cloud:** wait for webhook-driven cloud builds and verify via `tb_sys_push_build_verify`.
  - **Dream:** run **`dream inject push-specific`** for the relevant website/library repo(s), then verify via `tb_sys_push_build_verify`.
- Document notable env or build assumptions in the **context log** (`tb_sys_context_log`).
- For Go compile issues, validate handler code against **`tb_sdk_go_*`**.
