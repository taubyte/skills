---
description: Error handling - tau failures, Windows, git prohibition details
alwaysApply: false
---

# Taubyte Error Handling

## ERROR HANDLING

### Profile Switching Fails

IF `tau login <profile-name>` fails with "Profile does not exist": Create `tau login --new <profile-name>`. Verify `tau current`.

### Project Creation Fails

- Auth error: `tau login`, retry
- Name exists: Generate different snake_case name, retry
- Else: Check error, retry with corrected params
- Retry: Up to 2 times, 2-3s between

### Clone Fails

- Auth error: `tau login`, retry
- Project doesn't exist: For Dream/Local, ensure dream is started and universe exists first (see taubyte-dream), then `tau list projects`; retry
- Else: Check error, retry
- Retry: Up to 2 times

### Resource Creation Fails

- Missing flags: Check `.cursor/{resource}-commands.md`, include ALL flags, retry
- Windows path conversion: Prefix with `MSYS_NO_PATHCONV=1` (e.g. `MSYS_NO_PATHCONV=1 tau new function --paths /api/notes ...`) or `export MSYS2_ARG_CONV_EXCL='--paths'` in .bashrc
- Else: Log, skip resource, continue, inform user
- Special: Database needs `--match`, `--min`, `--max`, `--size`. Always include `--min`, `--max`, `--size` for database creation (e.g. `--min 1 --max 1 --size 1GB`). Function needs `--timeout`, `--memory`. Never `--timeout 0`.

### Build Fails

1. `tau query logs --jid [job-id]`
2. Analyze, fix code/config
3. Push again, verify `tau query builds`

### Push Fails - NEVER Use Git Workaround

- NEVER: git push, git commit, git add, git init, git remote add, git checkout -b (except creating main after clone)
- ALWAYS fix root cause of tau failure

**Decision Tree:** Check error → verify project (`tau current`), resource exists, config exists → IF uncommitted changes: commit/stash first → IF resource not found: push config, retry → IF auth: tau login → Report error, no git bypass. Retry up to 2 times.

### Windows File Locking (Access Denied)

**Problem:** tau push fails with "Access is denied" or "rename ... failed" - file locks from antivirus, IDE, etc.

**Retry:** Wait 2-5s, retry. Max 3 attempts with backoff (2s, 4s, 8s). IF fails: Check open files, antivirus; wait 5-10s; retry. IF still fails: Report error; consider `dream inject push-all` for Dream. DO NOT use git push.

### Tau Command Failure Protocol

NEVER bypass tau with git. Check: project selected, resource exists, config exists. IF config missing: Recreate or manual config. Retry. IF fails: Check auth, universe. Report error. Fix root cause.

**Validation after resource creation:** Pre: tau current. Post: config exists, resource in list, config pushed. IF any fail: Fix before code generation.

## WINDOWS-SPECIFIC RULES

### [MUST] Path Flag Handling

**Problem:** Git Bash converts `--paths` to Windows paths.

**Solution:** `MSYS_NO_PATHCONV=1 tau new website --paths / ...` or `MSYS_NO_PATHCONV=1 tau new function --paths /api/notes ...`. Or permanently: `export MSYS2_ARG_CONV_EXCL='--paths'` in .bashrc. Path is domain-relative, NOT filesystem.

**Affected:** `tau new website --paths`, `tau new function --paths`

### Hosts File Editing (Dream/Local)

After creating domains: Add to Windows hosts. Requires admin. See taubyte-dream Hosts File Protocol.

## GIT BRANCH MANAGEMENT

- All repos use `main`. Use `--branch main` for clones.
- After clone: Verify `git branch --show-current` → main. IF not: `git checkout -b main` or `git fetch && git checkout main`.
- Repos: tb_config_*, tb_code_*, tb_website_*, tb_library_*
