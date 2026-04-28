---
name: tb_sys_cloud_selection
description: Selects cloud context. If no FQDN is provided, always use Dream local and ensure Dream is started.
---

# Cloud Selection

## Rule

- If user provides FQDN -> remote cloud.
- If no FQDN provided -> Dream local by default.
- For Dream/local, do not run backend-contacting `tau` commands until Dream/universe readiness is confirmed.

## Steps

1. Inspect:
   ```bash
   tau --defaults --yes json current
   ```
2. Remote mode:
   ```bash
   tau select cloud --fqdn <cloud_fqdn>
   tau --defaults --yes json current
   ```
   - If the next operation includes `tau clear application`, immediately re-check context and re-select remote FQDN if drifted:
     ```bash
     tau --defaults --yes clear application
     tau --defaults --yes json current
     tau --defaults --yes select cloud --fqdn <cloud_fqdn>
     ```
3. Dream default mode:
   ```bash
   dream status universe default
   ```
4. If Dream status fails or not running:
   ```bash
   # If you have newer Dream:
   dream start
   # If you have stable/older Dream (no `dream start`):
   dream new multiverse --daemon --universes default
   dream new universe default
   dream status universe default
   ```
5. Select Dream cloud:
   ```bash
   tau select cloud --universe default
   tau --defaults --yes json current
   ```

## Remote context hard check

- For any remote mutation (`tau push`, `tau import`, `tau new domain --generated-fqdn`, `tau delete domain`), context must show:
  - `Cloud Type: remote`
  - `Cloud: <cloud_fqdn>`
- If not, stop and re-select the cloud before proceeding.

## Dream readiness protocol

1. Confirm Dream is reachable (`dream status universe default`).
2. If unavailable, start Dream:
   - Newer Dream: `dream start`
   - Stable/older Dream: `dream new multiverse --daemon --universes default`
   Then create/select `default` universe.
3. Only after readiness is confirmed, continue with list/query/select operations that contact backend services.

## Dream deployment prerequisites (local)

- Before `dream inject push-specific`, ensure Docker is running:
  ```bash
  docker info
  ```
- If Docker is not available on Windows, start Docker Desktop and re-check.
- Record Dream/Docker readiness state in `tb_sys_context_log`.
