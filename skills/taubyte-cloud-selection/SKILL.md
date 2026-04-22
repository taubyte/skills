---
name: taubyte-cloud-selection
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
   tau --json current
   ```
2. Remote mode:
   ```bash
   tau select cloud --fqdn <cloud_fqdn>
   ```
3. Dream default mode:
   ```bash
   dream status universe default
   ```
4. If Dream status fails or not running:
   ```bash
   dream start
   dream new universe default
   dream status universe default
   ```
5. Select Dream cloud:
   ```bash
   tau select cloud --universe default
   tau --json current
   ```

## Dream readiness protocol

1. Confirm Dream is reachable (`dream status universe default`).
2. If unavailable, start Dream and create/select `default` universe.
3. Only after readiness is confirmed, continue with list/query/select operations that contact backend services.

## Dream deployment prerequisites (local)

- Before `dream inject push-all` / `push-specific`, ensure Docker is running:
  ```bash
  docker info
  ```
- If Docker is not available on Windows, start Docker Desktop and re-check.
- Record Dream/Docker readiness state in `taubyte-context-log`.
