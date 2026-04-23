# Taubyte Agent Skills

Reusable [Agent Skills](https://agentskills.io/) for Taubyte, structured as a workflow-first pack inspired by [`vercel-labs/skills`](https://github.com/vercel-labs/skills).

## What This Repo Provides

- A full Taubyte execution workflow (auth -> cloud select -> scope -> resource ops -> push/verify).
- Safety constraints to reduce invalid commands, bad configs, and deployment mistakes.
- Local Dream helpers, including host-based launch for `*.localtau` resources.
- Optional deployment support for self-hosted cloud via `@taubyte/spore-drive`.

## Prerequisites

- **Node.js** (includes `npx`)
- **Git** (the installer clones this repository)

You do **not** need a Vercel account or the `vercel` CLI.

## Quick Start (Recommended)

Install globally for Cursor (copy-ready):

```bash
mkdir -p ~/.cursor/skills
npx skills add owner/repo --agent cursor --global --copy --yes
```

Example for this repo:

```bash
mkdir -p ~/.cursor/skills
npx skills add taubyte/skills --agent cursor --global --copy --yes
```

## Install By Major IDE/Agent

Each block below is copy-ready for one editor/agent.

### Cursor

```bash
npx skills add taubyte/skills --agent cursor --global --copy --yes
```

### Claude Code

```bash
npx skills add taubyte/skills --agent claude-code --global --copy --yes
```

### Codex

```bash
npx skills add taubyte/skills --agent codex --global --copy --yes
```

### GitHub Copilot

```bash
npx skills add taubyte/skills --agent github-copilot --global --copy --yes
```

### Windsurf

```bash
npx skills add taubyte/skills --agent windsurf --global --copy --yes
```

If your `npx` cache is stale, use:

```bash
npx skills@latest add taubyte/skills --agent cursor --global --copy --yes
```

## Common CLI Tasks

See [skills CLI](https://github.com/vercel-labs/skills) for full docs.

| Goal | Command |
|------|---------|
| List skills in this repo (no install) | `npx skills add taubyte/skills --list` |
| Install one skill globally for Cursor | `npx skills add taubyte/skills --skill taubyte-core-rules --agent cursor --global --copy --yes` |
| Install all skills globally for Cursor | `npx skills add taubyte/skills --agent cursor --global --copy --yes` |
| Template command for any repo | `npx skills add owner/repo --agent cursor --global --copy --yes` |
| Create global Cursor skills path | `mkdir -p ~/.cursor/skills` |

## Skill Catalog

| Skill | Purpose |
|-------|---------|
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

## Required Workflow Order

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
11. Environment-specific + recovery as needed

## Private Forks

If you fork this repo as **private**, ensure Git authentication is configured (SSH or HTTPS credentials), same as any private `git clone`.

## License

MIT — see [LICENSE](LICENSE).
