---
name: taubyte-resource-flags-reference
description: Enriched resource command/flag reference combining legacy docs style with current CLI behavior, for both non-interactive and interactive modes.
---

# Resource Flags Reference

Use this as the canonical flag catalog to avoid duplicating long flag lists in other skills.

## Mode policy

- Default: non-interactive (`--defaults --yes` + explicit flags)
- Optional: interactive (no `--defaults`) by user request

## Common create commands (non-interactive)

```bash
tau --defaults --yes new domain --name <domain> --fqdn <fqdn> --type auto --description "<desc>"
tau --defaults --yes new database --name <db> --match <matcher> --min 1 --max 2 --size 1GB
tau --defaults --yes new storage --name <storage> --match <matcher> --size 1GB --bucket Object --public
tau --defaults --yes new messaging --name <msg> --match <matcher> --mqtt --ws
MSYS_NO_PATHCONV=1 tau --defaults --yes new service --name <svc> --protocol /api
MSYS_NO_PATHCONV=1 tau --defaults --yes new function --name <fn> --type http --domains <domain> --method GET --paths /api/x --source . --call Handler --template empty --language Go --timeout 30s --memory 64MB
MSYS_NO_PATHCONV=1 tau --defaults --yes new website --name <site> --domains <domain> --paths / --template empty --generate-repository --no-private --no-embed-token --branch main
MSYS_NO_PATHCONV=1 tau --defaults --yes new library --name <lib> --path /libraries/<lib> --template empty --generate-repository --no-private --no-embed-token --branch main
```

## Domain generation guardrails

- Never add `--generated-fqdn-prefix` by default.
- Only use `--generated-fqdn-prefix <prefix>` when the user explicitly requests a specific prefix.
- If the user does not request a prefix and generated FQDN is needed, use:

```bash
tau --defaults --yes new domain --name <domain> --generated-fqdn --type auto --description "<desc>"
```

## Function post-create guardrail

- Immediately after `new function --template empty`, replace scaffold content in `<function_path>/empty.go` with an actual handler implementation before push/test.

## Repository binding guardrails (website/library)

- Never call `new website`/`new library` without explicit repository strategy.
- Preferred deterministic creation:

```bash
MSYS_NO_PATHCONV=1 tau --defaults --yes new website \
  --name <site> \
  --domains <domain> \
  --paths / \
  --template empty \
  --generate-repository \
  --repository-name tb_code_<project>_<site> \
  --private \
  --no-embed-token \
  --branch main
```

- If using existing repository, pass explicit `--repository-name` or `--repository-id`.
- After creation, run import before push:

```bash
tau import website --name <site>
tau import library --name <lib>
```

- After creation/import, verify repo fullname/ID in generated config YAML before any push.

## Build/runtime config references

- Use `taubyte-build-runtime-config` for `.taubyte/build.sh` and `.taubyte/config.yaml` management.
- Add runtime variables under `environment.variables` (or resource-equivalent schema) when server-side config is required.
- For websites, build script must place deployable output in `/out`.

## Legacy-style interactive note

- In interactive mode, prompts are expected for visibility, template choice, embed-token, path/domain/method selections, and confirmation.
