---
name: tb_sys_push_build_verify
description: Pushes config/code and verifies builds/logs. Includes website/library push handling with tau command first, git fallback.
---

# Push Build Verify

## Steps

0. **Context:** Run **`tau --defaults --yes json current`**. If the user asked for a **new** project in this session, confirm **`Project`** matches that name (see **`tb_sys_router`** → *New project hard gate*) before any push or import.

1. Pre-push build/runtime config check:
   - Validate `.taubyte/build.sh` and `.taubyte/config.yaml` for changed functions/websites/libraries.
   - Ensure required env vars are declared when server-side config is needed.
   - For **Go functions**, confirm sources follow **`tb_sdk_go_*`**: **`empty.go` at function root only** (no manual `lib/`, no hand-written `main.go`).
   - **Never** build, run, curl, or “test” functions/libraries locally unless the user explicitly asks for a local build/run. Default is **GitHub push → cloud build** (Dream uses inject; remote uses webhook).
2. Push to GitHub (required):
   - **Config repo** (`tb_config_<project>`): `git add -A && git commit -m "<message>" && git push`
   - **Code repo** (`tb_code_<project>`): `git add -A && git commit -m "<message>" && git push`
   - **Website/library repos** (if changed): `git add -A && git commit -m "<message>" && git push`
3. **Remote cloud build (default):**
   - After GitHub push, **wait for webhook-driven builds** in the cloud.
   - Verify progress via `tau query builds` / `tau query logs` when available.
4. **Dream build trigger (local only):**
   - After GitHub push, run **`dream inject push-specific`** for the relevant website/library repos.
   - Then verify via `tau query builds` / `tau query logs`.
5. **Wait for builds to finish (required):**

   Website/library pushes can depend on resources being present in config. After a config push, **do not** push website/library code until the **latest Config** build is **✔ success** (or fails).

   ```bash
   # poll until the latest Config row shows ✔ (success) or × (failed)
   tau --defaults --yes query builds --since 1h

   # if failed, inspect logs and fix config before continuing
   tau --defaults --yes query logs --jid <config_job_id>
   ```

   - Remote caveat (`aventr.cloud`): if build table is empty even after confirmed push, verify via GitHub webhook deliveries and cloud console before declaring failure.

6. Check jobs/logs (capture IDs/diagnosis in context log):
   ```bash
   tau --defaults --yes query builds --since 1h
   tau --defaults --yes query logs --jid <job_id>
   ```
7. Re-check builds/logs and update context log.

## Push policy guardrails

- **Primary** workflow is `git push` to GitHub; cloud builds are triggered from GitHub (webhook or Dream inject).
- If the user explicitly insists on using `tau push ...`, treat it as an alternative interface to publishing, but keep the same policy: **no local run/build**, and **Dream uses inject push-specific only** unless explicitly requested otherwise.
- Always record relevant build/job IDs and latest log diagnosis in context log after checks.
- If Patrick reports `project ids not equal <patrick> != <yaml>`, align `config/config.yaml` `id:` with the cloud canonical project id before re-pushing config.
