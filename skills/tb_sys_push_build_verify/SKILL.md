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
   - For **Go functions**, confirm sources follow **`tb_sdk_go_*`**: **`empty.go` at function root only** (no manual `lib/`, no hand-written `main.go`). Optional local WASM check: **`taubyte/go-wasi`** Docker recipe in that skill.
2. If website/library was newly created in this workflow, import first:
   ```bash
   tau import website --name <website>
   tau import library --name <library>
   ```
   - Run only for resources that exist and are relevant to this workflow.
3. Push project config:
   ```bash
   tau --defaults --yes push project --config-only --message "<message>"
   ```
4. **Wait for the Config build to finish (required):**

   Website/library pushes can depend on resources being present in config. After a config push, **do not** push website/library code until the **latest Config** build is **✔ success** (or fails).

   ```bash
   # poll until the latest Config row shows ✔ (success) or × (failed)
   tau --defaults --yes query builds --since 1h

   # if failed, inspect logs and fix config before continuing
   tau --defaults --yes query logs --jid <config_job_id>
   ```

   - Remote caveat (`aventr.cloud`): if build table is empty even after confirmed push, verify via GitHub webhook deliveries and cloud console before declaring failure.

5. Check jobs/logs (capture IDs/diagnosis in context log):
   ```bash
   tau --defaults --yes query builds --since 1h
   tau --defaults --yes query logs --jid <job_id>
   ```
6. Push project code:
   ```bash
   tau --defaults --yes push project --code-only --message "<message>"
   ```
7. Website/library repositories (**only after config build success**):
   - Preferred:
     ```bash
     tau --defaults --yes push website --name <website> --message "<message>"
     tau --defaults --yes push library --name <library> --message "<message>"
     ```
   - Exception-only fallback for website/library repos:
     ```bash
     git add .
     git commit -m "<message>"
     git push origin main
     ```
8. Re-check builds/logs and update context log.

## Push policy guardrails

- For project config/code (`tau push project`), never bypass failures with git; fix tau/root cause first.
- Git fallback is allowed only for website/library repo-local push path when tau website/library push is unavailable or unsupported.
- If git fallback is used, record reason in context log.
- Always record relevant build/job IDs and latest log diagnosis in context log after checks.
- If Patrick reports `project ids not equal <patrick> != <yaml>`, align `config/config.yaml` `id:` with the cloud canonical project id before re-pushing config.
