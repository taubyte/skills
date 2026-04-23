# Taubyte Agent Skills

Reusable [Agent Skills](https://agentskills.io/) for Taubyte, structured in a modular workflow-first style inspired by [`vercel-labs/skills`](https://github.com/vercel-labs/skills).

## Prerequisites

- **Node.js** (includes `npx`)
- **Git** (the installer clones this repository)

You do **not** need a Vercel account or the `vercel` CLI.

## Install

From your project directory (recommended, installs all skills in one step):

```bash
npx skills add taubyte/skills --all
```

Or with a full URL:

```bash
npx skills add https://github.com/taubyte/skills --all
```

### Options (see [skills CLI](https://github.com/vercel-labs/skills))

| Goal | Example |
|------|---------|
| List skills without installing | `npx skills add taubyte/skills --list` |
| Install one skill | `npx skills add taubyte/skills --skill taubyte-core-rules` |
| Install all skills (non-interactive) | `npx skills add taubyte/skills --all` |
| Install all skills for Cursor only | `npx skills add taubyte/skills --skill "*" -a cursor -y` |
| Non-interactive | `npx skills add taubyte/skills -y` |
| Cursor only | `npx skills add taubyte/skills -a cursor` |
| Global (user-wide) | `npx skills add taubyte/skills -g` |

## Skills in this repo (fresh workflow pack)

| Skill | Description |
|-------|-------------|
| `taubyte-orchestrator` | Master gatekeeper: login -> mode -> cloud -> scope -> resources -> push/verify. |
| `taubyte-auth-and-profile` | Profile/login setup and validation. |
| `taubyte-execution-modes` | Non-interactive default + optional interactive mode. |
| `taubyte-core-constraints` | Non-negotiable Taubyte rules and failure-prevention constraints. |
| `taubyte-cloud-selection` | Dream-by-default when no FQDN, remote when FQDN is provided. |
| `taubyte-scope-routing` | Global scope vs application scope routing. |
| `taubyte-project-and-application` | Project selection and conditional application selection. |
| `taubyte-resource-flags-reference` | Full command/flag catalog for creation flows. |
| `taubyte-resource-creation` | Scope-aware resource creation patterns. |
| `taubyte-go-sdk-constraints` | Must-use go-sdk signatures/patterns for HTTP, PubSub, and Storage. |
| `taubyte-build-runtime-config` | `.taubyte/build.sh` + `.taubyte/config.yaml` + env var server-side configuration logic. |
| `taubyte-reference-index` | Topic index into deep legacy references for on-demand loading. |
| `taubyte-context-log` | Auto-maintains per-project context log file to reduce hallucinations. |
| `taubyte-push-build-verify` | Config/code/website/library push + build/log verification. |
| `taubyte-dream-local-operations` | Local Dream lifecycle and inject workflows. |
| `taubyte-hosts-file-editor` | Safe, idempotent hosts-file updates for local `*.localtau` hostname access. |
| `taubyte-local-host-launch` | Launch and verify Dream local resources by FQDN/port in curl and browser. |
| `taubyte-remote-cloud-operations` | Remote cloud deployment workflow. |
| `taubyte-debugging-and-recovery` | Failure diagnosis and recovery checklist. |
| `taubyte-spore-drive-sdk` | Self-hosted Taubyte cloud deployment via `@taubyte/spore-drive`. |

## Mandatory order

1. `taubyte-auth-and-profile`
2. `taubyte-execution-modes` (default: non-interactive)
3. `taubyte-core-constraints`
4. `taubyte-cloud-selection`
5. `taubyte-scope-routing`
6. `taubyte-project-and-application`
7. `taubyte-context-log` (initialize/update log)
8. `taubyte-resource-creation`
9. `taubyte-build-runtime-config` (when needed)
10. `taubyte-push-build-verify`
11. environment-specific + recovery as needed

## Private repository

If you fork this repo as **private**, ensure Git can authenticate (SSH or HTTPS with credentials), same as any `git clone`.

## License

MIT — see [LICENSE](LICENSE).
