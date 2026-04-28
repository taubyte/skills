---
name: tb_sys_remote_cloud
description: Remote cloud workflow with explicit fqdn selection and deployment verification.
---

# Remote Cloud Operations

## Steps

1. Select remote cloud and verify context:
   ```bash
   tau select cloud --fqdn <cloud_fqdn>
   tau --defaults --yes json current
   ```
2. Select project and verify:
   ```bash
   tau --defaults --yes select project --name <project_name>
   tau --defaults --yes json current
   ```
3. If `tau clear application` is used, verify cloud did not drift to Dream:
   ```bash
   tau --defaults --yes clear application
   tau --defaults --yes json current
   ```
4. Run resource and push workflows (non-interactive):
   ```bash
   tau --defaults --yes push project --config-only --message "<message>"
   tau --defaults --yes push project --code-only --message "<message>"
   ```
5. For website/library push, select app and push with message:
   ```bash
   tau --defaults --yes select application --name <app>
   tau --defaults --yes push website --name <website> --message "<message>"
   tau --defaults --yes push library --name <library> --message "<message>"
   ```
6. Verify build jobs and logs:
   ```bash
   tau --defaults --yes query builds --since 1h
   tau --defaults --yes query logs --jid <job_id>
   ```
7. If `tau query builds` is empty on remote, verify via webhook + console:
   - GitHub webhook deliveries for the pushed repository.
   - Taubyte console jobs/builds for `<cloud_fqdn>`.
8. Update context log.

## Notes

- If config build fails with `project ids not equal <patrick> != <yaml>`, align `config/config.yaml` `id:` with the cloud canonical project id and re-push config.
- Domain FQDN may be accepted at creation time but fail later if DNS/TXT proof is missing.
- Do not invent `*.default.<cloud_fqdn>` unless DNS is explicitly delegated.
- For generated domains on remote, confirm `Cloud Type: remote` first, then run:
  ```bash
  tau --defaults --yes new domain --name <name> --generated-fqdn --type auto --description "<desc>"
  ```
- **Never** use **`--generated-fqdn-prefix`**, **`--g-prefix`**, or any synonym on **`tau new domain`** (unconditional — see **`tb_sys_resource_flags_reference`**).
- After `new function --template empty`, always update generated `empty.go` before push/runtime tests.
