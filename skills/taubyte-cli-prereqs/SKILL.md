---
name: taubyte-cli-prereqs
description: Hard preflight for Node.js, tau/dream installation, version checks, and first-time GitHub login. Must run before any Taubyte workflow.
---

# Taubyte CLI Prerequisites (Hard Gate)

## Rule

Run this gate before any other Taubyte skill.

1. If **Node.js** is missing, **stop** and **ask the user** to install it (do not run `npm`).
2. If **`tau login` / GitHub auth** has never been completed for this machine, **stop** and **ask the user** to run **`tau login`** in a terminal and finish **browser GitHub sign-in** before any repo or cloud work.
3. If `tau` or `dream` is missing after Node is OK, install CLIs. If install fails, stop immediately.

## Step 0: Node.js (required for npm-installed `tau` / `@taubyte/dream`)

Detect:

```bash
command -v node >/dev/null 2>&1 && node --version
command -v npm >/dev/null 2>&1 && npm --version
```

If `node` is missing or `node --version` fails:

1. **Stop** the Taubyte workflow immediately.
2. **Ask the user** to install **Node.js** (LTS recommended), for example:
   - Windows: https://nodejs.org/ or `winget install OpenJS.NodeJS.LTS`
   - macOS: https://nodejs.org/ or `brew install node`
   - Linux: distro Node package or https://nodejs.org/
3. Tell them to **open a new terminal** after install, then rerun this skill from Step 0.

Do **not** run `npm i -g @taubyte/cli` or `npm i -g @taubyte/dream` when Node is absent.

## Step 1: First-time GitHub login (`tau login`)

Any workflow that touches **GitHub** (clone, `tau new project`, `tau push`, generated repos) needs an authenticated Taubyte profile.

1. Run **`tau --json current`** and inspect **Profile** (and cloud selection as needed).
2. If there is **no profile**, or the user says they have **never** logged in on this machine, **stop** and **ask the user** to run:

   ```bash
   tau login
   ```

   That starts **browser-based GitHub authentication** (OAuth / device flow depending on CLI version). The user must **complete the flow in the browser** themselves. Do not skip this step or invent tokens.

3. After they confirm login, verify again:

   ```bash
   tau --json current
   ```

4. For multiple profiles or named setup, use **`taubyte-auth-and-profile`** (`tau login --new --name <profile> --provider github`, etc.).

If the user cannot or will not complete **`tau login`**, **do not continue** to project create, push, import, or website repo operations.

## Step 2: detect `tau` / `dream` binaries and versions

```bash
command -v tau >/dev/null 2>&1 && tau version
command -v dream >/dev/null 2>&1 && (dream --version || dream --help) || true
```

If `dream` fails but Node works, a broken shell **alias** may point to a missing `dream.exe`. Prefer:

```bash
node "$(npm root -g)/@taubyte/dream/index.js" --help
```

## Step 3: install missing CLI(s)

Only when Steps 0–2 show Node is present, the user is logged in (or login not required yet for read-only steps), and `tau` or Dream is still missing.

Use `npm` as default cross-platform installer:

```bash
npm i -g @taubyte/cli
npm i -g @taubyte/dream
```

Alternative installer (when npm path is blocked):

```bash
curl https://get.tau.link/cli | sh
curl https://get.tau.link/dream | sh
```

## Step 4: verify install after changes

```bash
node --version
tau version
dream --version || dream --help || node "$(npm root -g)/@taubyte/dream/index.js" --help
```

## Hard stop condition

- If **Node** is missing after asking the user to install: stop.
- If **GitHub auth** is required for the next step but **`tau login`** was not completed: stop and ask the user to run **`tau login`** and finish in the browser.
- If `tau` or Dream still fails after install attempts: stop.
- Log the failure and attempted fixes in `.taubyte_ai/logs.txt`.
- Do not continue to cloud/project/resource operations.
