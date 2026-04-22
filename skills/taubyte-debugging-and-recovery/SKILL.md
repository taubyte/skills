---
name: taubyte-debugging-and-recovery
description: Recovery playbook for context drift, build failures, Dream issues, and path-conversion errors.
---

# Debugging And Recovery

## Checklist

1. Verify context:
   ```bash
   tau --json current
   ```
2. Verify resources:
   ```bash
   tau --json list projects
   tau --json list applications
   ```
3. Build diagnostics:
   ```bash
   tau query builds --since 1h
   tau query job --jid <job_id>
   tau query logs --jid <job_id>
   ```
4. Windows path fallback:
   ```bash
   MSYS_NO_PATHCONV=1 <command>
   ```
5. Dream diagnostics:
   ```bash
   dream status universe default
   dream status gateway default
   ```
6. Update context log with failure + fix summary, including latest job IDs and diagnosis.

## Go compile guardrail

- If failures include assignment mismatch/undefined methods in go-sdk calls, apply `taubyte-go-sdk-constraints` before retrying.
