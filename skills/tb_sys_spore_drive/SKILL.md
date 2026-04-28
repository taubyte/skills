---
name: tb_sys_spore_drive
description: Deploy/self-host Taubyte cloud with @taubyte/spore-drive SDK using code-first automation and minimal required user input.
---

# Spore Drive SDK

Use this skill only when user asks to deploy/self-host a Taubyte cloud across servers.

## Inputs to collect

- Root domain (`ROOT_DOMAIN`)
- Generated subdomain (`GENERATED_DOMAIN`)
- Server list (`hostname,public_ip`)
- SSH key path (`SSH_KEY`)
- Tau binary mode (`latest` or custom path)
- Optional DNS automation choice

## SSH user policy

- Try `ubuntu` first.
- If deployment fails with SSH auth/user mismatch, retry with `root`.
- If both fail, ask user for SSH username.

## Implementation flow

1. Create deployment project (e.g. `spore-drive-deploy/`).
2. Generate SDK deployment code using `@taubyte/spore-drive`.
3. Write `.env`, `.env.example`, `hosts.csv`.
4. Install/run:
   ```bash
   npm install
   npm run displace
   ```
5. Use headless log progress lines (no interactive progress bars).

## DNS guidance fallback

- If DNS is not automated, provide manual provider-agnostic records:
  - `seer` A records
  - `tau` NS records
  - wildcard CNAME to substrate/expected endpoint

## Notes

- Keep this skill isolated from normal project/app/resource workflows.
- See upstream `@taubyte/spore-drive` package docs when extending deployment code.
