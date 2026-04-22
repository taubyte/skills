---
name: taubyte-push-build-verify
description: Pushes config/code and verifies builds/logs. Includes website/library push handling with tau command first, git fallback.
---

# Push Build Verify

## Steps

1. Pre-push build/runtime config check:
   - Validate `.taubyte/build.sh` and `.taubyte/config.yaml` for changed functions/websites/libraries.
   - Ensure required env vars are declared when server-side config is needed.
2. Push project config:
   ```bash
   tau push project --config-only --message "<message>"
   ```
3. Check jobs/logs:
   ```bash
   tau query builds --since 1h
   tau query logs --jid <job_id>
   ```
4. Push project code:
   ```bash
   tau push project --code-only --message "<message>"
   ```
5. Website/library repositories:
   - Preferred:
     ```bash
     tau push website --name <website> --message "<message>"
     tau push library --name <library> --message "<message>"
     ```
   - Exception-only fallback for website/library repos:
     ```bash
     git add .
     git commit -m "<message>"
     git push origin main
     ```
6. Re-check builds/logs and update context log.

## Push policy guardrails

- For project config/code (`tau push project`), never bypass failures with git; fix tau/root cause first.
- Git fallback is allowed only for website/library repo-local push path when tau website/library push is unavailable or unsupported.
- If git fallback is used, record reason in context log.
- Always record relevant build/job IDs and latest log diagnosis in context log after checks.
