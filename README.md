# Taubyte Agent Skills

Reusable [Agent Skills](https://agentskills.io/) for Taubyte — install with the open-source [`skills` CLI](https://github.com/vercel-labs/skills) (`npx skills`). Works with Cursor, Claude Code, Codex, OpenCode, and [many other agents](https://github.com/vercel-labs/skills#supported-agents).

## Prerequisites

- **Node.js** (includes `npx`)
- **Git** (the installer clones this repository)

You do **not** need a Vercel account or the `vercel` CLI.

## Install

From your project directory:

```bash
npx skills add taubyte/skills
```

Or with a full URL:

```bash
npx skills add https://github.com/taubyte/skills
```

### Options (see [skills CLI](https://github.com/vercel-labs/skills))

| Goal | Example |
|------|---------|
| List skills without installing | `npx skills add taubyte/skills --list` |
| Install one skill | `npx skills add taubyte/skills --skill taubyte-core-rules` |
| Non-interactive | `npx skills add taubyte/skills -y` |
| Cursor only | `npx skills add taubyte/skills -a cursor` |
| Global (user-wide) | `npx skills add taubyte/skills -g` |

## Skills in this repo

| Skill | Description |
|-------|-------------|
| `taubyte-default-context` | Assume Taubyte by default unless the user opts out. |
| `taubyte-core-rules` | Critical automation, `tau`/`dream` rules, naming, matcher, layout. |
| `taubyte-reference-docs` | Command/logic/SDK reference markdown (`references/*.md`). |
| `spore-drive-sdk` | Deploy Taubyte cloud with `@taubyte/spore-drive`. |

## Private repository

If you fork this repo as **private**, ensure Git can authenticate (SSH or HTTPS with credentials), same as any `git clone`.

## License

MIT — see [LICENSE](LICENSE).
