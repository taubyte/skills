---
description: Deployment - push, build wait, URL provision, website code
globs: "**/tb_website_*/**,**/tb_code_*/**"
alwaysApply: false
---

# Taubyte Deployment

## WEBSITE CODE GENERATION

### Step 1: Clone Website

`tau clone website --name [name] --no-embed-token --branch main`

Verify branch: `cd tb_website_[name] && git branch --show-current` → main. IF detached/no branch: `git checkout -b main` or `git fetch && git checkout main`.

### Step 2: Tech Stack Decision

- Simple static: HTML/CSS/JS vanilla
- Dynamic data: Vue.js (preferred) or React
- Database: Vue.js + Taubyte database client
- File uploads: HTML form + fetch + storage client
- Complex state: Vue.js composition API

### Step 3: Generate Code Structure

- ALWAYS create `.taubyte` folder with proper structure
- Create `index.html`, JS/Vue files, `styles.css`
- Add database/storage integration via Taubyte clients
- NO TEMPLATES: Generate from scratch

### Step 4: Code Principles

- Modern ES6+, async/await
- try/catch error handling
- Responsive CSS (mobile-first)
- Clean, readable structure

## DEPLOYMENT

### Step 0: Uncommitted Changes Validation

**Before ANY push** (`tau push project`, `tau push website`, `tau push library`):

1. Check: `cd [project]/[repo-type] && git status --porcelain` (repo-type = config or code)
2. IF non-empty output: FAIL push, show which repo has uncommitted changes, DO NOT proceed
3. IF empty: Proceed

**VALIDATE:** No uncommitted changes before pushing

### Step 0.5: Build Wait Protocol

**After ANY push that triggers builds:**

1. Poll `tau query builds --json` until completion (max 10 min, 5s interval)
2. **Status 2** = Success; **Status 1** = Failed (exit, query logs); **Status 0** = Pending (continue)
3. IF any build fails: Exit, display error, `tau query logs --jid [job-id]`
4. IF timeout: Warning, show status, DO NOT proceed

Build mapping: config-only→Config build; code-only→Code build; website push→Website build; push-all→all; push-specific→specific.

**VALIDATE:** All expected builds Status 2 before proceeding

### Step 1: Push Website Code

`tau push website --name [name] --message "Initial website code"`

Wait for build (Build Wait Protocol). IF fails: stop. IF succeeds: proceed.

### Step 2.5: Dream—push-specific for websites [MUST]

**When deploying to Dream (local):** After `dream inject push-all --universe [name]` succeeds, **MUST** run push-specific for **each** website. Website builds are not triggered by push-all alone.

1. For each website: read `[project]/config/websites/[name].yaml`
2. Use `source.github.id` (numeric) for `--rid`, `source.github.fullname` for `--fn`
3. Run: `dream inject push-specific --universe [universe-name] --rid [source.github.id] --fn [source.github.fullname]`
4. **Do NOT use** the Taubyte resource ID (Qm...) for `--rid`—only the numeric GitHub repo ID.

Then run Build Wait Protocol for website build(s).

### Step 3: Remote Environment Deployment

After push: GitHub webhook triggers build. Wait 5-10s, then wait for builds (Build Wait Protocol). Proceed to URL provision when complete.

### Step 4: URL Provision

**After successful deployment:**

1. **Query domains:** `tau query website --name [name]` - Extract domain names. Skip if fails or none.
2. **For each domain:** `tau query domain --name [domain-name]` - Get FQDN. Extract path from config (default `/`). Format: `https://[fqdn][path]`
3. **Provide URLs:** "Website deployed successfully! Access at: [URL]" - List all. IF none: "Check domain configuration."

**VALIDATE:** URL(s) provided after deployment
