---
name: taubyte-core-rules
description: Critical Taubyte rules — automation, tau CLI, Dream/local, naming, database/storage matcher, HTTP functions, code layout, and git vs tau. Use for any Taubyte project or resource work.
---

# Taubyte Core Rules

Use this skill together with **taubyte-reference-docs** for command flags and SDK details.

## When to use

- Creating or modifying Taubyte projects, functions, websites, databases, storage, Dream deployments, or multiverse resources.
- Before running `tau`, `dream`, or changing generated config.

## Quick reference

1. Do not ask the user to run setup commands manually — automate setup.
2. For Dream/local: ensure Dream is running and the default universe exists before backend `tau` calls; use `dream` commands directly, not `tau dream`.
3. After resource changes: `tau push project --config-only`; after code changes: `tau push project --code-only` and verify builds.
4. Use `tau new` with full flags; use `--template empty` for functions and websites (Go: add `--language Go` with empty template).
5. One HTTP method per function — never comma-separated methods on one function.
6. Use **matcher** values from config for database/storage in code, not resource filenames.
7. Code layout: `<app_name>/functions/<name>/` at the **code repo root** (not only under `applications/`).
8. Do not use raw `git` to work around `tau` failures — fix the underlying issue.

## Full rules

Open the files under `references/` in this skill for the complete rule set (ported from Cursor project rules):

- `references/taubyte-core.md` — main checklist and principles
- `references/taubyte-dream.md`, `references/taubyte-deployment.md`, `references/taubyte-error-handling.md`, `references/taubyte-reference.md`, `references/taubyte-resources.md` — specialized rule docs
