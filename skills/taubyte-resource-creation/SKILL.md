---
name: taubyte-resource-creation
description: Scope-aware resource creation workflow. Uses non-interactive mode by default and references the shared flags catalog.
---

# Resource Creation

## Preconditions

- `taubyte-core-constraints` loaded
- `taubyte-scope-routing` completed
- `taubyte-project-and-application` completed
- `taubyte-context-log` initialized
- `taubyte-build-runtime-config` available for server-side config/build logic

## Order

- Global resources first (domain, website, library)
- Application resources next (database, storage, messaging, service, function, smartops)

## Rules

- Use default non-interactive mode unless user requested interactive.
- For full flags, use `taubyte-resource-flags-reference`.
- Use `MSYS_NO_PATHCONV=1` for path-like flags in Git Bash.
- Never use `--generated-fqdn-prefix` unless the user explicitly asks for a specific prefix value.
- If server-side config or env vars are needed, update `.taubyte/config.yaml` and `.taubyte/build.sh` before push.
- For Go function implementation correctness, apply `taubyte-go-sdk-constraints`.
- After creating a serverless function with `--template empty`, immediately edit the generated `empty.go` with a real implementation (do not leave scaffold code).
- Website/library creation must use explicit repo strategy; never rely on implicit repo auto-selection.
- Deterministic default for generated repos:
  - website: `tb_code_<project>_<website>`
  - library: `tb_code_<project>_<library>`
- Immediately import after creation to sync local repository clone:
  - `tau import website --name <site>`
  - `tau import library --name <lib>`
- Immediately verify bound repo fullname/ID in config yaml after creation/import; if mismatch, stop and fix before any push.
- Update context log after every mutation.
