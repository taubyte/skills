---
name: deploy-taubyte-spore-drive
description: >-
  Guides deployment of a self-hosted Taubyte (Tau) cloud using the
  spore-drive-boilerplate (clone, hosts.csv, .env, npm install, npm run displace)
  and explains manual DNS records when Namecheap API vars are omitted. Use when
  the user wants to deploy Taubyte on their own servers with Spore Drive, AI-assisted
  setup of spore-drive-boilerplate, or post-deploy DNS for tau/seer/generated domains.
---

# Deploy Taubyte cloud (Spore Drive boilerplate)

End-to-end workflow matches the upstream repo:
https://github.com/taubyte/spore-drive-boilerplate

Execute steps in order. Prefer running commands in the user’s environment rather than only describing them.

## Prerequisites

- Node.js and npm
- Linux servers with SSH on port 22, sudo-capable user, key-based SSH
- A DNS zone the user controls for `ROOT_DOMAIN` (and parent zone if `ROOT_DOMAIN` is a subdomain)

## 1. Clone the boilerplate

```bash
git clone https://github.com/taubyte/spore-drive-boilerplate.git
cd spore-drive-boilerplate
```

## 2. Create `hosts.csv`

Path defaults to `hosts.csv` (overridable via `SERVERS_CSV_PATH`).

Format (header row required; column names must match):

- `hostname` — FQDN of each node (e.g. `node1.mycloud.com`)
- `public_ip` — public IPv4 of that node

One row per Taubyte node; multi-node clouds add multiple rows.

Example:

```csv
hostname,public_ip
node1.mycloud.com,203.0.113.1
node2.mycloud.com,203.0.113.2
```

## 3. SSH key path

In `.env`, set `SSH_KEY` to the **filesystem path** of the **private** key used to reach every host (same key as the boilerplate README; not the pubkey).

Ensure permissions are restrictive (e.g. `chmod 600` on the key file) if SSH complains.

## 4. Environment file (ask the user; skip Namecheap)

```bash
cp .env.example .env
```

Collect values conversationally and write `.env`. **Do not require or prompt for Namecheap variables** (`NAMECHEAP_API_KEY`, `NAMECHEAP_IP`, `NAMECHEAP_USERNAME`, `NAMECHEAP_DOMAIN`) unless the user explicitly wants API-managed DNS.

Variables to set (mirror `.env.example` / README):

| Variable | Purpose |
|----------|---------|
| `SSH_KEY` | Path to SSH private key |
| `SERVERS_CSV_PATH` | CSV path (default `hosts.csv`) |
| `SSH_USER` | SSH login user (default `ssh-user`) |
| `ROOT_DOMAIN` | Platform root domain (e.g. `example.com`) |
| `GENERATED_DOMAIN` | Generated subdomain for the platform; typically `g.<ROOT_DOMAIN>` or another label under `ROOT_DOMAIN` |

Convention from README: `GENERATED_DOMAIN` should be a subdomain of `ROOT_DOMAIN` (e.g. `ROOT_DOMAIN=pom.ac`, `GENERATED_DOMAIN=g.pom.ac`). If Namecheap vars are **all** empty/unset, `src/index.ts` **skips** API DNS and the user must add records manually (see below).

## 5. Install and deploy

```bash
npm install
npm run displace
```

`displace` runs `tsx src/index.ts`: applies Spore Drive config, displaces the course, then runs `fixDNS`. With no Namecheap credentials, expect `[Skip] DNS Records` — that is expected when using manual DNS.

## 6. Manual DNS records (logic from `src/index.ts`)

When Namecheap vars are not all set, `fixDNS` returns false and **no** records are pushed. Reproduce the same logical records in the user’s DNS dashboard.

Derived from `fixDNS` (and the `zoneLines` it builds):

### 6.1 Names used in the simple case

Assume the DNS zone apex **is** `ROOT_DOMAIN` (e.g. `ROOT_DOMAIN=example.com` and the user manages the `example.com` zone).

Let:

- `generatedPrefix` = label part of `GENERATED_DOMAIN` before `.ROOT_DOMAIN`  
  Example: `GENERATED_DOMAIN=g.example.com` → `generatedPrefix=g`.

If `GENERATED_DOMAIN` does not end with `.${ROOT_DOMAIN}`, the script uses the full `GENERATED_DOMAIN` string as `generatedPrefix` — prefer the README pattern `g.<ROOT_DOMAIN>` to avoid ambiguity.

### 6.2 Records to create

1. **A — `seer.<ROOT_DOMAIN>`**  
   One **A** record **per** node IP: same name (`seer` / `seer.<ROOT_DOMAIN>`), each **address** is a distinct `public_ip` from hosts that participate in shape `all` (in practice, each row in `hosts.csv` after a successful deploy). Multiple A records on the same name match Spore-drive’s `setAll(seerHost, "A", seerAddrs)` behavior.

2. **NS — `tau.<ROOT_DOMAIN>`**  
   Nameserver target: **`seer.<ROOT_DOMAIN>`** (must be the hostname that has the A records above). In zone-file form the target appears as `seer.<ROOT_DOMAIN>.` (FQDN).

3. **CNAME — `*.<generatedPrefix>.<ROOT_DOMAIN>`**  
   Example: `*.g.example.com` when `generatedPrefix=g`.  
   Target (alias): **`substrate.tau.<ROOT_DOMAIN>`** (e.g. `substrate.tau.example.com`).

### 6.3 When `ROOT_DOMAIN` is not the zone apex

If `ROOT_DOMAIN` is a **subdomain** of the registered zone (e.g. `ROOT_DOMAIN=tau.example.com` while the registrar zone is `example.com`), the **relative** host labels in the boilerplate shift:

- `subPrefix` = part of `ROOT_DOMAIN` before `.<apex>` (e.g. `tau` for `tau.example.com` under apex `example.com`).
- **A**: names like `seer.<subPrefix>` under zone **apex** → e.g. `seer.tau.example.com`.
- **NS**: `tau.<subPrefix>` under apex → e.g. `tau.tau.example.com` → NS target `seer.<ROOT_DOMAIN>.` (i.e. `seer.tau.example.com.`).
- **CNAME**: `*.` + `generatedPrefix` + `.` + `subPrefix` under apex → e.g. `*.g.tau.example.com` → target `substrate.tau.<ROOT_DOMAIN>.`.

If unsure which case applies, resolve the zone apex (SOA for `ROOT_DOMAIN`) or ask which domain is the **registrar / DNS provider zone** vs **platform** subdomain.

### 6.4 What to tell the user

After a successful `npm run displace`, output a short checklist:

- List each **A** for `seer` with the concrete IPs from their `hosts.csv` (or from committed host addresses if known).
- The **NS** on `tau.<ROOT_DOMAIN>` pointing at `seer.<ROOT_DOMAIN>`.
- The **wildcard CNAME** for `*.<generatedPrefix>.<ROOT_DOMAIN>` → `substrate.tau.<ROOT_DOMAIN>`.
- TTL: upstream uses **1800** seconds where applicable; any reasonable TTL is fine.

Remind them to add records in the **zone that actually serves** `ROOT_DOMAIN` (registrar or DNS host).

## Optional: Namecheap automation

If the user later sets all of `NAMECHEAP_USERNAME`, `NAMECHEAP_API_KEY`, and `NAMECHEAP_IP`, re-running the script’s DNS step can push records automatically; `NAMECHEAP_DOMAIN` is optional (SOA discovery otherwise). This skill’s default path is **manual DNS** without those variables.

## Troubleshooting pointers

- CSV errors: column names must be exactly `hostname` and `public_ip` (`src/csv.ts`).
- SSH: `SSH_USER` must match a user with sudo on every host; `SSH_KEY` must be readable.
- Displacement errors: progress output lists failing host/task pairs.
- DNS skip: empty Namecheap env trio → `[Skip] DNS Records`; add manual records from section 6.
