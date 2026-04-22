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
- If server-side config or env vars are needed, update `.taubyte/config.yaml` and `.taubyte/build.sh` before push.
- For Go function implementation correctness, apply `taubyte-go-sdk-constraints`.
- Update context log after every mutation.
