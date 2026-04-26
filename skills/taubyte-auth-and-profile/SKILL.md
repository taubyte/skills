---
name: taubyte-auth-and-profile
description: Handles Taubyte login/profile setup and first-time browser GitHub auth via tau login --new when no account exists; tau login for existing profiles. Stops immediately when browser login is required. Uses non-interactive login when the user supplies a GitHub username; otherwise asks for it explicitly. Must run before cloud/project/resource operations.
---

# Auth and profile

Use this skill before **cloud, project, resource, clone, push, or GitHub-backed** `tau` work. The user must be signed in to **GitHub through the Tau CLI** where that work depends on it.

---

## Stop conditions (apply first)

**Browser-based GitHub login**

If login **requires the browser** (no usable **`--token`**, or `tau` still sends the user to the browser):

1. **Tell the user** they must finish GitHub authorization in the browser (the agent cannot do it for them).
2. Give the exact **`tau login …`** command to run when that helps.
3. **Stop the workflow immediately.** Do **not** continue with other `tau` steps, pushes, clones, or repo operations until the user **confirms** login finished.
4. Then re-check with **`tau --json current`**.

**User will not authenticate**

If GitHub-backed work is needed but the user will not run **`tau login --new`** (no account / first machine) or **`tau login`** / **`tau login <profile_name>`** (existing account) as appropriate: **stop** and explain that **`tau new project`**, **`tau push`**, clone, and repo registration **will fail** without auth.

---

## Policy: profile name, token, and non-interactive mode

**What `tau login` actually accepts** (confirm with **`tau login --help`** on the installed binary):

| Flag | Short | Role |
|------|--------|------|
| `--token` | `-t` | Git provider token (e.g. GitHub PAT) for non-browser login when available |
| `--name` | `-n` | Tau **profile** name |
| `--provider` | `-p` | Provider (default **`github`**) |
| `--new` | — | Create a new profile; implied when no profiles exist |
| `--set-default` | `-d` | Set as default profile (`--no-set-default` / `--no-d` to disable) |

Usage line: **`tau login [command options] <name>`** — profile label may be **`--name` / `-n`** or the trailing **`<name>`** positional.

**Globals** (from **`tau --help`**) go **before** the subcommand, e.g. **`tau --defaults --yes login …`**, per **`taubyte-execution-modes`** when automating.

There is **no** separate “GitHub username” flag on `login`. Use **`--name`** for the **Tau profile name** (often the same string as the GitHub handle). Use **`--token`** when you have a PAT for unattended login.

**GitHub handle / profile name**

- **User already gave a GitHub handle** (or explicit profile name): prefer **`tau --defaults --yes login …`** where applicable; set **`--name`** to the name they asked for, or their handle as **`<profile_name>`** if they did not specify otherwise; add **`--token`** when a token is available. Do not re-prompt unless unclear or wrong.
- **User did not give a handle** when you need it for **`--name`** or repo context: **ask in chat** before suggesting or running commands. Do not guess or invent a profile name.

**Token vs browser**

- **With** a valid **`--token`**, a browser step is **usually** unnecessary.
- **Without** **`--token`**, expect **browser** GitHub sign-in → **Stop conditions** above.

---

## Workflow

### 1. Check current context

```bash
tau --json current
```

### 2. If profile is missing or there is no Tau account yet

Standard path: **`tau login --new`** with GitHub (not optional for first account). Apply **Policy** and **Stop conditions** before choosing flags.

**Example (explicit provider and profile name):**

```bash
tau login --new --provider github --name <profile_name>
```

- If the user should **only attach or switch** an existing account: **`tau login`** or **`tau login <profile_name>`** (positional profile per **`tau login --help`**).
- If you are **automating** and have a handle plus PAT: **`tau --defaults --yes login …`** with **`--token`** and **`--name`** as documented in help.

If this path **does not** use **`--token`**, treat it as **browser OAuth** → **Stop conditions**: inform the user, **stop**, resume only after they confirm, then go to step 3.

### 3. After the user confirms login (including any browser step)

```bash
tau --json current
```

Do not treat login as complete until the user has confirmed and this check shows a valid profile as expected.

---

## Named profile and switching

Creating a named profile uses the same **`tau login --new … --name <profile_name>`** pattern as in step 2.

```bash
tau login --new --provider github --name <profile_name>
```

Switching default / context to an existing profile:

```bash
tau login <profile_name>
```

Apply the same **Policy** and **Stop conditions** as for the first-time flow.

---

## Quick verify

```bash
tau --json current
```

Use after login and whenever you need to confirm which profile is active.
