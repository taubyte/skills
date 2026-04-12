---
description: Dream/Local env - multiverse, Docker, hosts file, deployment
globs: "**/tb_*/**"
alwaysApply: false
---

# Taubyte Dream Environment

## ENVIRONMENT SETUP

### Environment Detection Decision Tree

```
IF user mentions "locally", "local", "dream", "test locally", "local testing", "run locally"
  → THEN Dream/Local environment
ELSE
  → THEN Remote environment (default)
```

### Dream Environment Setup

**[MUST] Multiverse Startup Protocol:**

**[MUST]** NEVER run `tau list projects` or other tau commands that contact the backend (e.g. list/query) until multiverse is running AND the target universe exists. If in doubt, run the full setup (dream start → dream new universe default if missing) first.

1. **Check Status:** `dream status` OR check for running process
2. **IF multiverse NOT running:**
   - Start in background: `dream start` (runs continuously)
   - Wait 5-10 seconds for initialization
   - **VALIDATE:** `dream status universe` - check exit code and output
   - **IF connection refused:** Wait 5 more seconds and retry; if still fails, wait up to 30 seconds total
   - **NEVER** prompt user to start manually
3. **IF multiverse IS running:** Proceed directly to universe setup
4. **After multiverse confirmed running:**
   - Check `tau current` for default universe
   - **IF default universe does NOT exist AND user did NOT request new:** Create with `dream new universe`, wait 2-3s, select `tau select cloud --universe default`
   - **IF default universe EXISTS:** `tau select cloud --universe default`
   - **IF user requests new universe:** Create `dream new universe [name]`, wait 2-3s, select cloud
5. **Only after this** may you run `tau list projects`, `tau current`, or other tau commands that hit the auth/config backend.

**[MUST] Universe Usage Rule:** NEVER create new universe per project unless explicitly told. ALWAYS use default "default".

**[MUST] Dream Command Usage:** Use `dream` commands directly, NEVER `tau dream`.

### Remote Environment Setup

1. Check `tau current`
2. IF cloud not selected: `tau select cloud` (select "remote", enter FQDN)
3. IF auth needed: `tau login`
4. **VALIDATE:** `tau current` shows correct cloud and authentication

### GitHub Profile Handling (if username provided)

IF profile matches: Proceed. ELSE: Try `tau login <github-username>`. IF "Profile does not exist": `tau login --new <github-username>`. **VALIDATE:** `tau current` shows correct Profile.

## [MUST] Hosts File Editing Protocol (Dream/Local Only)

**After successfully creating a domain in Dream/Local environment:**

1. **Verify Environment:** `tau current` - Skip if remote or not local
2. **Query Domain:** `tau query domain --name [domain-name]` - Extract FQDN. Skip if query fails, FQDN not found, or FQDN doesn't end with `.localtau`
3. **Parse FQDN:** Format `[prefix].g.[universe].localtau`. Skip if format doesn't match
4. **Prompt User:** Ask for admin permissions to add to hosts file
5. **Read hosts file:** `C:\Windows\System32\drivers\etc\hosts`
6. **Check existing:** Skip if entry already exists
7. **Add entry:** `# Taubyte Local Domains` marker if needed, then `127.0.0.1 [fqdn]`
8. **Write:** Use elevated permissions. On failure, show manual instructions.

**VALIDATE:** Entry added or user declined; no duplicates; points to 127.0.0.1

**User self-service:** For step-by-step instructions so the user can edit the hosts file alone (Windows + Linux/macOS), point them to `.cursor/dream-hosts-instructions.md`.

## Docker Engine Verification (Dream Environment Only)

1. **Check:** `docker info` (or `docker info > /dev/null 2>&1` for scriptability). If exit non-zero, Docker is not running or not reachable.
2. **IF NOT running (Windows):** Start Docker Desktop with: `start "" "C:\Program Files\Docker\Docker\Docker Desktop.exe"`. Wait for it to load (e.g. poll `docker info` every few seconds until it succeeds, or wait ~30–60 seconds then retry). If still not running after a reasonable wait, then ask the user to start Docker and retry.
3. **IF NOT running (non-Windows):** Ask the user to start the Docker daemon, then retry.
4. **VALIDATE:** Only proceed with deployment (e.g. push-all, push-specific) when `docker info` succeeds.

## Dream Environment Deployment

**PREREQUISITE:** Docker must be running.

1. Wait for any prior code/config builds to complete (Build Wait Protocol)
2. Extract universe: `tau current`
3. **Run push-all (REQUIRED first):** `dream inject push-all --universe [universe-name]`
4. Wait for push-all builds. **CRITICAL:** push-specific will NOT work until push-all succeeds
5. **[MUST] After push-all succeeds:** IF the project has websites, run **push-specific for EACH website** (do not skip):
   - Get **numeric GitHub repo ID** from config: `[project]/config/websites/[name].yaml` → `source.github.id` (e.g. `1157540083`). **--rid must be this number, NOT the Taubyte resource ID (Qm...).**
   - Get repo fullname: same file → `source.github.fullname` (e.g. `ghir-hak/tb_website_notenow_web`)
   - Run: `dream inject push-specific --universe [universe-name] --rid [source.github.id] --fn [source.github.fullname]`
   - Repeat for each website in the project. Wait for website build(s).

**[MUST]** Always specify `--universe` for both commands. **push-all MUST run first and MUST succeed** before push-specific. **Never omit push-specific for websites**—website builds are triggered by push-specific, not push-all alone.
