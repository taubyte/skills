---
name: enforcing-taubyte-constraints
description: Numbered catalog of the non-negotiable Taubyte rules — naming, push ordering, domain CLI-management, matcher consistency, function layout, build/runtime env placement, and Dream vs remote build triggers — with one-line statements and cross-links to the skill that owns each rule's deep coverage. Use as a final pre-push checklist, when reviewing a teammate's change, or when you need a single audit point that touches every "must / never" rule across the collection.
---

# Enforcing Taubyte Constraints

## When to use

- Final review pass before `tau push` / `git push`
- Auditing a session's actions ("did I cover all the hard rules?")
- Reviewing a teammate's PR against a Taubyte project
- Onboarding a new agent / engineer to the project

## How to use this skill

Each rule is a one-liner. The link points to the skill that explains and enforces the rule. Read the rule list top-to-bottom; if any rule is unclear or violated, jump to the linked skill before continuing.

## Architecture & sources of truth

1. **GitHub is the single source of truth.** Clouds (remote and Dream) build and serve from what's pushed; never bypass git. → [understanding-taubyte-architecture](../understanding-taubyte-architecture/SKILL.md)
2. **Config repo is `tau`-managed.** Don't hand-edit `config/...` YAML; mutations go through `tau new` / `tau edit` / `tau delete`. → [editing-taubyte-resources](../editing-taubyte-resources/SKILL.md)
3. **Domains are CLI-managed always.** Never `vim config/domains/<x>.yaml`; use `tau new domain` / `tau edit domain` / `tau delete domain`. → [creating-taubyte-resources](../creating-taubyte-resources/SKILL.md)

## Naming & validation

4. **Project name = snake_case.** Dashes break `tau validate config` with `invalid variable name`. → [bootstrapping-taubyte-projects](../bootstrapping-taubyte-projects/SKILL.md)
5. **`config/config.yaml` must have a real `notification.email`.** Empty values fail config-compile with `mail: no address`. → [bootstrapping-taubyte-projects](../bootstrapping-taubyte-projects/SKILL.md)
6. **Forbidden flag**: never pass `--generated-fqdn-prefix` / `--g-prefix` to `tau new domain`. Use `--generated-fqdn` alone or `--fqdn <controlled>`. → [creating-taubyte-resources](../creating-taubyte-resources/SKILL.md)
7. **Database `--min < --max`.** `--min 1 --max 1` fails. → [creating-taubyte-resources](../creating-taubyte-resources/SKILL.md)
8. **One function per `(path, method)`.** Never comma-separated methods; split into separate function resources. → [authoring-taubyte-function-types](../authoring-taubyte-function-types/SKILL.md)

## Context hygiene

9. **Verify `tau --json current` before any mutation** (push, delete, inject, `tau new` resource). The `Project` field must equal the on-disk project. → [selecting-taubyte-context](../selecting-taubyte-context/SKILL.md)
10. **After `tau new project`, immediately re-run `tau select project --name <same>` then verify.** Selection drift after `new project` is the usual cause of "wrong project" pushes. → [bootstrapping-taubyte-projects](../bootstrapping-taubyte-projects/SKILL.md)
11. **Clear application after app-scoped work**: `tau clear application` before returning to project-level resources. → [selecting-taubyte-context](../selecting-taubyte-context/SKILL.md)

## Code authoring rules

12. **Function root layout (Go)**: `empty.go` and `go.mod` at the function root. **Never** create a `lib/` subdirectory; never hand-author `main.go`. **Corollary:** careless **`tau build`** can still **generate** those paths as root-owned junk — **never commit them**; delete, `git rm --cached`, and `.gitignore` before push. → [writing-taubyte-functions](../writing-taubyte-functions/SKILL.md), [verifying-taubyte-functions](../verifying-taubyte-functions/SKILL.md)
13. **Package name is NOT `main`.** Use `lib` (or any other name). → [writing-taubyte-functions](../writing-taubyte-functions/SKILL.md)
14. **`//export <name>` must equal `execution.call`** in the function YAML. → [authoring-taubyte-function-types](../authoring-taubyte-function-types/SKILL.md)
15. **HTTP handler order**: `Headers().Set` → `Write(body)` → `Return(status)`, in that order. Returning before writing yields empty/broken responses. → [writing-taubyte-functions](../writing-taubyte-functions/SKILL.md)
16. **YAML matcher ↔ Go literal must match exactly**, including any leading `/`:
    - `databases/<x>.yaml` `match` ↔ `database.New("<match>")`
    - `storages/<x>.yaml` `match` ↔ `storage.New("<match>")`
    - `messaging/<x>.yaml` `channel.match` ↔ function `trigger.channel` ↔ `pubsubnode.Channel("<match>")`
    → [writing-taubyte-functions](../writing-taubyte-functions/SKILL.md)

## Build & runtime config

17. **Env vars live in `.taubyte/build.sh` only** as `export NAME=value`, never in `.taubyte/config.yaml`. → [configuring-taubyte-build-runtime](../configuring-taubyte-build-runtime/SKILL.md)
18. **Websites must write to `/out`.** Use the stack's real output directory; don't assume Vite (`dist/`) on a CRA repo (`build/`). → [building-taubyte-websites](../building-taubyte-websites/SKILL.md)
19. **`.taubyte/build.sh` must be non-empty and executable.** Empty scripts "build successfully" while producing nothing. → [configuring-taubyte-build-runtime](../configuring-taubyte-build-runtime/SKILL.md)

## Push & build triggers

20. **`--message` is required when `--defaults` is set.** Every `tau push` invocation in non-interactive mode must include `-m "<msg>"`. → [pushing-taubyte-projects](../pushing-taubyte-projects/SKILL.md)
21. **Push order**: project config first → wait for config build success → then code/website/library. → [pushing-taubyte-projects](../pushing-taubyte-projects/SKILL.md)
22. **Don't bypass a failing `tau push project` with raw `git push`** for the project's config/code repos. Website/library repos may use raw git as a fallback when `tau push website|library` is unavailable. → [pushing-taubyte-projects](../pushing-taubyte-projects/SKILL.md)

## Dream-local rules

23. **First-run bootstrap**: `dream inject push-all` once on a fresh project before `push-specific` becomes reliable. → [triggering-dream-builds](../triggering-dream-builds/SKILL.md)
24. **`push-specific` requires long flags + universe last** to avoid `Required flags "repository-id, repository-fullname" not set`. → [triggering-dream-builds](../triggering-dream-builds/SKILL.md)
25. **Register website/library repos with the local repository service** after `tau import website|library` (PUT `/repository/github/<repo_id>`); otherwise `push-specific` won't trigger builds for them. → [registering-dream-repositories](../registering-dream-repositories/SKILL.md)
26. **`dream inject` is Dream-only.** Never run it against a remote cloud — webhooks handle that side. → [deploying-to-remote-clouds](../deploying-to-remote-clouds/SKILL.md)
26.1. **Dream requires a real git HEAD in each injected repo.** Empty-branch repos (no commits) can make inject fail with `getting HEAD ... reference not found`. → [triggering-dream-builds](../triggering-dream-builds/SKILL.md)

## Remote-cloud rules

27. **Project-id alignment**: `config/config.yaml` `id:` must equal the cloud's canonical project id, or every push fails with `project ids not equal`. → [deploying-to-remote-clouds](../deploying-to-remote-clouds/SKILL.md)
28. **Generate domains only while `Cloud Type: remote`** (or the right Dream universe). The generated TLD is per-cloud. → [deploying-to-remote-clouds](../deploying-to-remote-clouds/SKILL.md)
29. **Use `--type https` (not `http`) for remote functions.** Remote gateways expect TLS. → [authoring-taubyte-function-types](../authoring-taubyte-function-types/SKILL.md)
30. **Remote website/library builds require registration + clean hooks.** If remote builds are silent: `tau import website|library <name>` on that cloud, ensure a single GitHub webhook (dedupe if needed), then push a commit and verify via job logs. → [deploying-to-remote-clouds](../deploying-to-remote-clouds/SKILL.md), [pushing-taubyte-projects](../pushing-taubyte-projects/SKILL.md)

## Platform / shell

31. **Git Bash on Windows**: prefix path-like flags with `MSYS_NO_PATHCONV=1` to avoid path mangling on `--paths`, `--protocol`, etc. → [creating-taubyte-resources](../creating-taubyte-resources/SKILL.md)
32. **Inline function disk path must match the active builder.** An extra `applications/` segment (or the inverse) makes builds fail with **`.taubyte` not found** under the path the job reports. Align `code/.../functions/<fn>/` to that path before changing anything else. → [creating-taubyte-resources](../creating-taubyte-resources/SKILL.md)
33. **PUT may not route like GET/DELETE** even when YAML declares PUT — the edge can return **`no HTTP match for method PUT`** while other methods work; logs may be incomplete. Prefer a **dedicated POST route** (e.g. `/api/resource/update`) as a separate function until routing is confirmed. → [authoring-taubyte-function-types](../authoring-taubyte-function-types/SKILL.md)
34. **Keep scaffold `empty.go`** (and matching `FILENAME` in `build.sh`) unless you have an image-verified recipe — renaming to `lib.go` / changing the entry file without full alignment breaks TinyGo and can drop **`artifact.wasm`**. → [configuring-taubyte-build-runtime](../configuring-taubyte-build-runtime/SKILL.md), [writing-taubyte-functions](../writing-taubyte-functions/SKILL.md)
35. **Website `.taubyte/build.sh` is executable shell only** — never paste HTML or other markup into it. A merged file breaks the script; replace with a minimal `mkdir -p /out` + copy pattern. → [building-taubyte-websites](../building-taubyte-websites/SKILL.md)
36. **Remote / public cloud: never ship `*.default.localtau` FQDNs** — the config compiler uses public DNS (`1.1.1.1`-class); Dream hostnames fail TXT checks. Delete + recreate domains via **`tau`** with a remote-suitable **`--fqdn`**. → [deploying-to-remote-clouds](../deploying-to-remote-clouds/SKILL.md), [editing-taubyte-resources](../editing-taubyte-resources/SKILL.md)
37. **Remote: `GET /` broken but `/api/...` OK** means the **website** repo artifact is missing — **`tau push website <site>`** + verify **`AssetCid`** on that repo’s build job, not further function debugging by default. → [pushing-taubyte-projects](../pushing-taubyte-projects/SKILL.md), [deploying-to-remote-clouds](../deploying-to-remote-clouds/SKILL.md), [building-taubyte-websites](../building-taubyte-websites/SKILL.md)
38. **`tau pull` → `already up-to-date` + non-zero exit** is often a CLI quirk — confirm with **`git fetch`/`git status`** before treating it as failure. → [pushing-taubyte-projects](../pushing-taubyte-projects/SKILL.md)
39. **Sibling dirs (`tau_notes` vs `tau_it`) + global `tau` selection**: **`tau --json current` → `Project`** must equal the **`config/config.yaml` `name:`** in the tree you `cd`’d into. → [selecting-taubyte-context](../selecting-taubyte-context/SKILL.md)

## Pre-push checklist (copy into the session)

```
Pre-push audit:
- [ ] tau --json current shows expected Cloud + Cloud Type + Project (rule 9)
- [ ] No raw YAML edits in config/ this session (rules 2, 3)
- [ ] Project name + matchers + channels are snake_case where required (rules 4, 16)
- [ ] If Go function: empty.go at root, package != main, //export matches call (rules 12, 13, 14)
- [ ] HTTP handlers: headers -> write -> Return(status) (rule 15)
- [ ] Env vars only in .taubyte/build.sh (rule 17); /out for websites (rule 18)
- [ ] tau push has -m "<message>" (rule 20)
- [ ] Push order: config first, then code/website/library (rule 21)
- [ ] Dream: bootstrap with push-all done, push-specific uses long flags + universe last (rules 23, 24)
- [ ] Remote: id alignment OK, no dream inject (rules 26, 27)
- [ ] If remote/Dream complained about `.taubyte` path: function dir matches log (rule 32); website build.sh is pure shell (rule 35)
- [ ] Remote: no lingering `localtau` domains in config when target is public FQDN (rule 36); if static page 500: website build + AssetCid verified (rule 37); `tau pull` quirks checked with git (rule 38); `tau` Project matches directory `config.yaml name` (rule 39)
```

## Related skills

- `understanding-taubyte-architecture` — why these rules exist
- `troubleshooting-taubyte-issues` (if added later) — what to do when one is violated
- The skill linked next to each rule above is the canonical owner for the rule's deep coverage
