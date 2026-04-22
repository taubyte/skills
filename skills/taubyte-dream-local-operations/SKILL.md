---
name: taubyte-dream-local-operations
description: Dream local lifecycle operations: start/status/universe, inject flows, and local validation.
---

# Dream Local Operations

## Steps

1. Ensure Dream is running and universe exists:
   ```bash
   dream status universe default
   dream start
   dream new universe default
   dream status universe default
   ```
2. Ensure Docker is running before inject/deploy:
   ```bash
   docker info
   ```
3. Select local cloud:
   ```bash
   tau select cloud --universe default
   tau --json current
   ```
4. Inject project (must run first):
   ```bash
   dream inject push-all --universe default --path <absolute_project_root>
   ```
5. For websites, run push-specific for each website after successful push-all:
   ```bash
   dream inject push-specific --universe default --rid <repo_id> --fn <owner/repo> --path <absolute_repo_path>
   ```
6. Validate and update context log.

## Critical rules

- Always pass `--universe`.
- `push-all` must succeed before `push-specific`.
- For website `push-specific`:
  - `--rid` must be numeric GitHub repository ID (`source.github.id`)
  - `--fn` must be repository fullname (`source.github.fullname`)
  - Do not use Taubyte Qm resource IDs for `--rid`.
- Use absolute local paths for `--path`.

## Hosts guidance for `.localtau`

- If a Dream domain resolves to `.localtau`, update hosts mapping when required.
- Typical mapping target: `127.0.0.1 <fqdn>`.
