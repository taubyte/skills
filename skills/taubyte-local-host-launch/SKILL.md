---
name: taubyte-local-host-launch
description: Launch and verify Dream local resources by hostname using hosts mappings and universe/substrate ports.
---

# Local Host Launch

## When to use

Use this skill after domain/resource creation in Dream/local when the goal is to:
- access website/function endpoints by FQDN
- open resources in browser using hostnames
- validate host-based routing

## Prerequisites

- `taubyte-dream-local-operations`
- `taubyte-hosts-file-editor`

## Steps

1. Confirm Dream and universe are running:
   ```bash
   dream status universe default
   ```
2. Get FQDN(s) from domain resources:
   ```bash
   tau query domain --name <domain-name>
   ```
3. Ensure hosts mappings exist for each FQDN:
   - `127.0.0.1 <fqdn>`
4. Discover serving port:
   - Usually from substrate:
     ```bash
     dream status substrate
     ```
   - If gateway is used in the universe, use its HTTP port from `dream status universe <name>`.
5. Validate by hostname:
   ```bash
   curl "http://<fqdn>:<port>/<path>"
   ```
6. If hosts edit is not available, fallback validation:
   ```bash
   curl --resolve "<fqdn>:<port>:127.0.0.1" "http://<fqdn>:<port>/<path>"
   ```
7. For browser launch:
   - open `http://<fqdn>:<port>/<path>`

## Notes

- Hosts mappings are OS-level and affect browser resolution.
- `curl --resolve` is request-scoped and does not modify system DNS.
- For HTTPS tests, prefer `--resolve` so hostname/SNI stays correct.
- Do not introduce `--generated-fqdn-prefix` unless the user explicitly asks for a prefix.
- If a function was created with `--template empty`, ensure `empty.go` is implemented before launching local tests.
