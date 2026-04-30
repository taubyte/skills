---
name: building-taubyte-websites
description: Authors a Taubyte website's `.taubyte/build.sh` so the cloud can serve it — the deploy output **must** be written to `/out`. Provides the minimal known-good static-HTML build script and the recommended `html` template baseline (or copying it into an existing site repo). Use when creating a website's build script for the first time, when a website builds but doesn't serve, or when migrating a site between Taubyte template baselines.
---

# Building Taubyte Websites

## When to use

- Creating a brand-new website repo and you need a working `.taubyte/build.sh`
- A website pushes and "builds" but the cloud serves nothing / 404s
- Adding a build script to a site that was generated with `--template empty`
- Migrating from another build to the Taubyte expected layout

## Hard rule

**The build script must write deployable static output into `/out`.** The Taubyte cloud serves whatever ends up in `/out` after `.taubyte/build.sh` finishes.

## Where the build script lives

```text
<website_repo>/
├── .taubyte/
│   ├── build.sh        # executed by the cloud build (and locally)
│   └── config.yaml     # build/runtime config (image, env, etc.)
└── ...                 # source files (index.html, src/, etc.)
```

## Minimal build.sh — static HTML

For a single-page or pure-static site whose source is already deploy-ready (e.g. `index.html` at the repo root):

```bash
#!/usr/bin/env bash
cp index.html /out
exit $?
```

That's it — no bundler, no build step. This is the pattern used by the bundled `html` website template and is the simplest known-good baseline.

## Recommended starting point — `html` template

When creating a new website, generate from the bundled `html` template so you start with a working `build.sh` + `index.html`:

```bash
tau --defaults --yes new website <site_name> \
  --domains <domain_name> \
  --paths /<path> \
  --template html \
  --generate-repository \
  --repository-name tb_code_<project>_<site_name> \
  --private \
  --no-embed-token \
  --branch main

tau --defaults --yes import website <site_name>
```

The template produces:

- An `index.html` with simple content
- `.taubyte/build.sh` that already does `cp index.html /out`

This is the recommended baseline before customizing.

## Migrating an existing `--template empty` site

If a site was created with `--template empty`, copy the `html` template's behavior into it:

1. Add `index.html` to the website repo root.
2. Create `.taubyte/build.sh` containing:
   ```bash
   cp index.html /out
   exit $?
   ```
3. Make it executable: `chmod +x .taubyte/build.sh`.
4. Push the website repo.

## Build script for stack-based sites (Vite / CRA / etc.)

The same rule holds: whatever the stack outputs must end up in `/out`.

Example shape for Vite:

```bash
#!/usr/bin/env bash
set -euo pipefail

npm install
npm run build              # writes ./dist/

mkdir -p /out
cp -a dist/. /out/
exit $?
```

Adapt the build command and source dir to your stack (CRA → `build/`, Vite → `dist/`, etc.). Declare environment variables in `.taubyte/build.sh` itself — that's the only place server-side build env lives.

## Verifying the build wrote to `/out`

When running locally (e.g. inside the Taubyte build container or via `bash .taubyte/build.sh` in a sandbox):

```bash
ls -la /out
```

Expect at least the entry document (e.g. `index.html`) and any required assets.

## Workflow checklist

```
Website build progress:
- [ ] index.html (or build artifacts) present in source repo
- [ ] .taubyte/build.sh exists and is executable
- [ ] build.sh writes everything served into /out
- [ ] tau push website <site> -m "..."
- [ ] For Dream: dream inject push-specific for the website repo
- [ ] curl through gateway with Host header to confirm content (see verifying-taubyte-functions)
```

If the gateway responds but content doesn't route (e.g. `no substrate match found`), retry the curl with a `Host` header that includes the gateway port:

```bash
curl -i -H "Host: <fqdn>:<gateway_port>" "http://127.0.0.1:<gateway_port>/"
```

## Gotchas

- **Empty `/out` after build = blank cloud** (or 404). Always confirm `cp` / `cp -a` / `mv` actually populated `/out`.
- **`.taubyte/build.sh` empty** is a common failure when a site was created with `--template empty` and never edited — it builds "successfully" but produces nothing.
- **Typos in `.taubyte/config.yaml` can silently break remote builds.** Example: `environment:` misspelled as `enviroment:` can cause the cloud builder to ignore config or fail early.
- **Stack-specific env**: don't put runtime env in `config.yaml`'s wrong section; declare build env in `build.sh` only.
- **Per-stack output dirs**: copy the **stack's output dir contents** to `/out`, not the dir itself. `cp -a dist/. /out/` is right; `cp -a dist /out/` puts files under `/out/dist/`.

## Related skills

- `creating-taubyte-resources` — `tau new website` flag patterns
- `pushing-taubyte-projects` — `tau push website`
- `triggering-dream-builds` — `dream inject push-specific` for the website repo
- `verifying-taubyte-functions` — also covers gateway curl with `Host:` header
