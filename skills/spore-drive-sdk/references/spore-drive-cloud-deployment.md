# Deploy a Taubyte Cloud (Spore Drive)

This guide describes how to deploy a self-hosted Taubyte cloud (PaaS/IDP) across your servers using [Spore Drive](https://github.com/taubyte/spore-drive-boilerplate) and [@taubyte/spore-drive](https://www.npmjs.com/package/@taubyte/spore-drive).

## Required Inputs

The AI infers what it can from the user's message (domain, subdomain, servers). **Prompt only for what's missing** — you are obliged to ask rather than fail silently. Use natural, conversational phrasing.

### Required Inputs

| Input | Description | Example |
|-------|-------------|---------|
| **Hostnames and public IPs** | List of servers: one `hostname,public_ip` pair per server | `node1.mycloud.com,203.0.113.1` |
| **SSH key path** | Path to SSH private key file | `ssh-key.pem` |
| **Root domain** | Root domain for your platform | `pom.ac` |
| **Generated domain** | Generated subdomain for your platform | `g.pom.ac` |

### SSH user — try ubuntu, then root, then ask

**Do not** ask for SSH user upfront. Use this order:

1. **Default to `ubuntu`** — set `SSH_USER=ubuntu` and run deployment.
2. **If deployment fails** (SSH auth/connection errors): retry with `SSH_USER=root`.
3. **If that also fails**: ask the user for their SSH username.

### Tau build (ask after required inputs)

Ask: **"Use the latest Tau build or a custom Tau binary? (latest / custom)"**

- **latest** — Use the default (no extra input). The boilerplate uses `TauLatest` from `@taubyte/spore-drive`.
- **custom** — Ask for the **path to your Tau binary** (e.g. `./tau` or `/usr/local/bin/tau`). Set `TAU_PATH` in `.env` and the boilerplate must use `TauPath(process.env.TAU_PATH)` instead of `TauLatest` when creating the Drive (see [spore-drive-boilerplate](https://github.com/taubyte/spore-drive-boilerplate): `new Drive(config, TauPath("path/to/tau"))`).

### Optional Input — DNS automation

After collecting the required values, ask: **"Do you want automated DNS configuration? (y/n)"**

- **If yes** — Automated DNS is supported for **Namecheap** (via API). Ask for: `NAMECHEAP_API_KEY`, `NAMECHEAP_IP`, `NAMECHEAP_USERNAME`. For other providers (Cloudflare, Route53, etc.), use manual DNS setup below.
- **If no** — Give the user **manual DNS instructions** for their provider (see below). Manual setup works with **any DNS provider** (Cloudflare, Route53, Namecheap, registrar panel, etc.).

### Manual DNS setup (any provider)

After `npm run displace` completes, provide **project-specific manual DNS instructions**. These work with any DNS provider (Cloudflare, Route53, Namecheap, etc.). Use the user's **ROOT_DOMAIN**, **GENERATED_DOMAIN**, and the public IPs from their **hosts.csv**. The generated prefix is the subdomain part of GENERATED_DOMAIN before the root (e.g. for `g.pom.ac` the prefix is `g`).

Give the user these instructions (fill in their values):

1. **seer.\<ROOT_DOMAIN\>** — Create **A** records, one per server public IP from their hosts.csv (e.g. `203.0.113.1`, `203.0.113.2`).
2. **tau.\<ROOT_DOMAIN\>** — Create one **NS** record pointing to **seer.\<ROOT_DOMAIN\>.** (include the trailing dot if their DNS provider requires it).
3. **\*.\<generatedPrefix\>** (wildcard for the generated domain) — Create a **CNAME** record pointing to **substrate.tau.\<ROOT_DOMAIN\>.** (e.g. for `g.pom.ac`, create `*.g` or the wildcard that covers the generated domain, CNAME to `substrate.tau.pom.ac.`).

The user creates these records in their DNS provider's dashboard (Cloudflare, Route53, Namecheap, or any registrar).

### [MUST] Paths in `.env` — TAU_PATH and SSH_KEY

Write **TAU_PATH** and **SSH_KEY** in `.env` exactly as the user provides them (the actual path). Do not convert to relative paths.

---

## AI / Taubyte extension flow (Cursor, etc.)

The AI **writes deployment code** using `@taubyte/spore-drive` (see the spore-drive-sdk skill). The user can provide domain, subdomain, and servers in one message; the AI extracts what it can. The AI **prompts for any missing required inputs** (SSH key, etc.) — obliged to ask, never fail silently. **SSH user:** use `ubuntu` first, then `root` if that fails; only ask the user for their SSH username if both fail. Create a deploy project (e.g. `spore-drive-deploy/`), generate `hosts.csv`, `.env`, and the deploy script; run `npm install` and `npm run displace` in the background. Use headless progress output (no TTY) for Cursor's embedded terminal, e.g. `[host] [uploading tau] [15%]`. If the user did **not** choose automated DNS (or uses a provider other than Namecheap), after displacement succeeds give them the **manual DNS instructions** for their provider (see "Manual DNS setup (any provider)" above).

---

## Getting Started (standalone / without extension)

### Step 1: Clone and Install

Clone or use the [spore-drive-boilerplate](https://github.com/taubyte/spore-drive-boilerplate) repository:

```bash
git clone https://github.com/taubyte/spore-drive-boilerplate.git
cd spore-drive-boilerplate
npm install
```

### Step 2: Create `hosts.csv`

Create `hosts.csv` (or use the path specified in `.env` as `SERVERS_CSV_PATH`) with the user-provided hostnames and public IPs:

**CSV format:**

```csv
hostname,public_ip
node1.mycloud.com,203.0.113.1
node2.mycloud.com,203.0.113.2
```

- `hostname`: Fully qualified domain name of the server
- `public_ip`: Public IP address of the server

### Step 3: Configure Environment

Copy the example environment file and fill it with the user-provided values:

```bash
cp .env.example .env
```

**`.env` variables (from user input):**

```bash
# Server Configuration
SSH_KEY=<user-provided-path>              # e.g. ssh-key.pem
SERVERS_CSV_PATH=hosts.csv               # Path to hosts.csv (default)
SSH_USER=ubuntu                         # default; use root if ubuntu fails; ask user only if both fail

# Domain Configuration
ROOT_DOMAIN=<user-provided>              # e.g. pom.ac
GENERATED_DOMAIN=<user-provided>         # e.g. g.pom.ac

# Tau binary (only if user chose custom build)
TAU_PATH=<path-to-tau-binary>           # e.g. ./tau or /usr/local/bin/tau; omit for latest

# Automated DNS (optional — Namecheap supported; other providers use manual setup)
NAMECHEAP_API_KEY=<if-using-namecheap-automation>
NAMECHEAP_IP=<if-using-namecheap-automation>
NAMECHEAP_USERNAME=<if-using-namecheap-automation>
```

### Step 4: Custom Tau binary (only if user chose custom)

If the user chose a custom Tau binary, the boilerplate must use it when creating the Drive. In `src/index.ts`, replace the Drive line with logic that uses `TauPath` when `TAU_PATH` is set:

- Default (latest): `const drive = new Drive(config, TauLatest);`
- Custom: `const drive = new Drive(config, process.env.TAU_PATH ? TauPath(process.env.TAU_PATH) : TauLatest);`

Ensure `TauPath` is imported from `@taubyte/spore-drive` (it is in the boilerplate).

### Step 5: Deploy

Run the deployment:

```bash
npm run displace
```

---

## Server Requirements

- Linux servers with SSH access on port 22
- User account (the SSH_USER value) with:
  - sudo/root privileges
  - SSH key authentication enabled (use the SSH_KEY for authentication)
