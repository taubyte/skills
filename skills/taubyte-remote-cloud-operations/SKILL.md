---
name: taubyte-remote-cloud-operations
description: Remote cloud workflow with explicit fqdn selection and deployment verification.
---

# Remote Cloud Operations

## Steps

1. Select remote cloud:
   ```bash
   tau select cloud --fqdn <cloud_fqdn>
   tau --json current
   ```
2. Run resource and push workflows.
3. Verify build jobs and logs:
   ```bash
   tau query builds --since 1h
   tau query logs --jid <job_id>
   ```
4. Update context log.

## Note

- Domain FQDN may be accepted at creation time but fail later in config build logs if DNS proof is missing.
- Do not use `--generated-fqdn-prefix` unless the user explicitly requests a prefix.
- After `new function --template empty`, always update generated `empty.go` before any push or runtime test.
