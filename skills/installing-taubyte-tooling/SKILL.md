---
name: installing-taubyte-tooling
description: Installs and verifies the prerequisites for any Taubyte workflow — Node.js, Docker (engine + running daemon), `@taubyte/cli` (the `tau` command), and `@taubyte/dream` (the `dream` command). Acts as a hard gate that runs before every other Taubyte skill, with OS-specific automated installs (winget / brew / apt-get) and explicit stop conditions when an install fails. Use on first-time machine setup, when `tau` or `dream` is missing or broken, when `docker info` fails, or when starting a fresh shell and you don't know if the toolchain is ready.
---

# Installing Taubyte Tooling

## When to use

- First-time machine setup for `tau` / `dream`
- `tau --version` / `dream --help` fails
- `docker info` fails or `docker` is missing
- `npm i -g @taubyte/cli` was attempted but didn't produce a working `tau`
- A new shell session and you don't know whether the toolchain is in place
- Hard gate **before** every other Taubyte skill (run this first if anything below seems off)

## Hard stop conditions

| Situation | Action |
| --- | --- |
| Node missing AND can't be installed from this environment | **Stop**; give OS-specific install instructions; do **not** run `npm i -g` until `node`/`npm` work. |
| Docker CLI missing AND daemon can't be started | **Stop** before Dream / `dream inject` / any container-backed flow. |
| `tau` or `dream` still broken after install attempts | **Stop**; record what was tried and the failure output. |
| GitHub-backed work needed but no `tau` profile | **Stop**; defer to [authenticating-taubyte-cli](../authenticating-taubyte-cli/SKILL.md). |

## Workflow

```
Prereq progress:
- [ ] Step 1: Node.js + npm work
- [ ] Step 2: Docker CLI present and `docker info` succeeds
- [ ] Step 3: `@taubyte/cli` installed → `tau --version` works
- [ ] Step 4: `@taubyte/dream` installed → `dream --help` works
- [ ] Step 5: Auth gate (only if GitHub work is in scope)
- [ ] Step 6: Final verification block runs clean
```

## Step 1 — Node.js + npm

### Detect

```bash
command -v node >/dev/null && node --version
command -v npm  >/dev/null && npm  --version
```

### Install when missing

| OS | Command |
| --- | --- |
| Windows (winget) | `winget install -e --id OpenJS.NodeJS.LTS --accept-package-agreements --accept-source-agreements` |
| macOS (brew) | `brew install node` |
| Linux (Debian/Ubuntu) | `sudo apt-get update && sudo apt-get install -y nodejs npm` |

If automated install fails, is blocked (no sudo / no winget/brew), or the OS is unclear, **stop** and tell the user to install Node.js LTS from `https://nodejs.org/`, then open a new terminal and rerun.

**Do not** run `npm i -g @taubyte/...` until both `node --version` and `npm --version` succeed.

## Step 2 — Docker (engine + running daemon)

Docker is required for Dream itself, `dream inject` flows, and local Go WASM verification (the `taubyte/go-wasi` recipe).

### Detect

```bash
command -v docker >/dev/null && docker version
docker info
```

Two distinct failure modes:

- `docker` not in PATH → treat as **not installed** (use install table below).
- `docker version` works but `docker info` fails with daemon/connection errors → daemon is **stopped** or you lack permission. Start the daemon, don't reinstall.

### Install when CLI is missing

| OS | Commands |
| --- | --- |
| Windows (winget) | `winget install -e --id Docker.DockerDesktop --accept-package-agreements --accept-source-agreements` (then start Docker Desktop once) |
| macOS (brew) | `brew install --cask docker` (then launch Docker Desktop) |
| Linux (Debian/Ubuntu) | `sudo apt-get update && sudo apt-get install -y docker.io && sudo systemctl enable --now docker` |

### Start the daemon when CLI is present

| OS | Command |
| --- | --- |
| Windows / macOS | Launch Docker Desktop, wait for the whale icon to indicate "running" |
| Linux | `sudo systemctl start docker` |
| Linux permission errors | `sudo usermod -aG docker $USER` then log out/in (or new shell) |

After starting, re-run `docker info` until it succeeds. If install/start is impossible (no admin, headless CI, blocked policy), **stop** and point the user to `https://docs.docker.com/get-docker/`.

## Step 3 — `tau` (`@taubyte/cli`)

### Detect

```bash
command -v tau >/dev/null && tau --version
tau --help    # confirm subcommand surface
```

### Install when missing

```bash
npm i -g @taubyte/cli
```

Alternative when the global npm path is blocked:

```bash
curl https://get.tau.link/cli | sh
```

## Step 4 — `dream` (`@taubyte/dream`)

### Detect

```bash
command -v dream >/dev/null && (dream --version || dream --help)
```

### Install when missing

```bash
npm i -g @taubyte/dream
```

Alternative:

```bash
curl https://get.tau.link/dream | sh
```

If `dream` is on PATH but errors immediately, the global wrapper may be broken. Run via the package directly to confirm:

```bash
node "$(npm root -g)/@taubyte/dream/index.js" --help
```

## Step 5 — Auth gate (only when GitHub work is in scope)

Any flow that creates/imports projects, pushes, or generates repos needs a working `tau` profile. **Do not improvise auth here.** Defer to [authenticating-taubyte-cli](../authenticating-taubyte-cli/SKILL.md).

Quick gate check:

```bash
tau --defaults --yes json current
```

Look at the `Profile` field. If empty and the next step touches GitHub, run [authenticating-taubyte-cli](../authenticating-taubyte-cli/SKILL.md) before continuing.

## Step 6 — Final verification

Run as a single block; expect every line to print a version/info line without errors:

```bash
node --version
npm  --version
docker version
docker info
tau --version
dream --version || dream --help
```

If any line fails, return to that step and re-do. Do **not** proceed to project / cloud / resource work with a partially-broken toolchain.

## CLI drift note

`tau` and `dream` evolve quickly. Don't assume a subcommand exists based on older docs. Confirm capability with `tau --help` / `dream --help` before scripting around it. See [starting-dream-locally](../starting-dream-locally/SKILL.md) for the `dream start` vs `dream new multiverse` split that depends on this.

## Gotchas

- **`npm i -g` without working `node`** silently no-ops or installs broken shims — always verify `node --version` first.
- **`docker version` succeeds, `docker info` fails** is a stopped daemon, not a missing install. Don't reinstall.
- **Linux permission errors** (`Got permission denied while trying to connect to the Docker daemon socket`) are fixed by `usermod -aG docker $USER` + logout/login, not by `sudo docker` long-term.
- **`tau` or `dream` errors immediately** with cryptic `node`/`require` errors → likely a broken global npm install. Reinstall (`npm i -g @taubyte/cli` / `@taubyte/dream`) or use the `curl https://get.tau.link/...` script.
- **Don't gate on a single command's exit code** for the verification block — read each line; some commands print version on success but exit non-zero on `--version`-vs-`--help` flag mismatch.

## Related skills

- `authenticating-taubyte-cli` — runs after this gate when GitHub work is in scope
- `starting-dream-locally` — first thing you do once `dream` is installed
- `understanding-taubyte-architecture` — context for why `dream` + `docker` are needed at all
- `verifying-taubyte-functions` — uses the `taubyte/go-wasi` Docker image; depends on Docker daemon
