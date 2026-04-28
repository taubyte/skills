---
name: tb_sys_local_host_launch
description: Launch and verify Dream local resources by hostname using hosts mappings and universe/substrate ports.
---

# Local Host Launch

## When to use

Use this skill **only when the user explicitly requests local execution/verification** (curl, browser open, hostname routing). Default Taubyte workflow is **not** to run anything locally.

## Prerequisites

- `tb_wf_dream_inject_and_verify`
- `tb_sys_hosts_file`

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
   - do not modify hosts automatically unless the user explicitly requests it
4. Ensure Dream build trigger is executed for Dream/local (no local build/run):
   ```bash
   docker info
   dream inject push-specific --universe default ...
   ```
   - If the user did not explicitly ask for local verification, stop after inject + build/log verification (do not curl/open).
5. Discover serving port:
   - Usually from substrate:
     ```bash
     dream status substrate
     ```
   - If gateway is used in the universe, use its HTTP port from `dream status universe <name>`.
6. If (and only if) the user explicitly asked to validate locally, provide **commands** rather than executing them:
   - `curl "http://<fqdn>:<port>/<path>"`
   - or `curl --resolve "<fqdn>:<port>:127.0.0.1" "http://<fqdn>:<port>/<path>"`
   - browser URL: `http://<fqdn>:<port>/<path>`

## Output contract

- Always return:
  - exact website URL(s),
  - exact function URL(s),
  - one ready-to-run curl per endpoint (only when local verification was explicitly requested),
  - whether hosts mapping requires manual admin edit (only when the user asked for hosts-based access).

## Notes

- Hosts mappings are OS-level and affect browser resolution.
- `curl --resolve` is request-scoped and does not modify system DNS.
- For HTTPS tests, prefer `--resolve` so hostname/SNI stays correct.
- **Never** pass **`--generated-fqdn-prefix`** / **`--g-prefix`** to **`tau new domain`** (unconditional ban across all Taubyte skills).
- If a function was created with `--template empty`, ensure `empty.go` is implemented before launching local tests.
