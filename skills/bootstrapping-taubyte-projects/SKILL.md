---
name: bootstrapping-taubyte-projects
description: Bootstraps a Taubyte project on the currently selected cloud — `tau new project` to create config + code repos on GitHub and clone them locally, `tau import project` (with repo fullnames, not paths) to register an existing pair, and `tau --json current` to verify selection. Encodes the snake_case project-name rule (dashes break `tau validate config` with `invalid variable name`). Use when creating, importing, or cloning a Taubyte project for the first time.
---

# Bootstrapping Taubyte Projects

## When to use

- "Create a new Taubyte project"
- "Import an existing project into this cloud"
- `tau validate config` fails with `/config.yaml:2:7: invalid variable name`
- `tau` is on a cloud but doesn't see the project you expect
- First-time project setup on Dream or remote

## Snake_case naming rule (critical)

`tau` validates the project name as a YAML variable inside the config repo. **Use `snake_case` (or `lowercase_alnum_underscores`) — never dashes.**

| Project name | Outcome |
| --- | --- |
| `my_project` | OK — `tau validate config` passes |
| `team_app_v2` | OK |
| `my-project` | FAILS — `/config.yaml:2:7: invalid variable name` |

If you (or a teammate) created a project with dashes, the cleanest fix is to recreate it under a snake_case name and re-import.

## Preconditions

- `tau` authenticated to GitHub (see [authenticating-taubyte-cli](../authenticating-taubyte-cli/SKILL.md)).
- A cloud is selected (Dream local or remote) — see [selecting-taubyte-context](../selecting-taubyte-context/SKILL.md).
- For Dream: the universe is up — see [inspecting-dream-status](../inspecting-dream-status/SKILL.md).

## A — Create a brand-new project

`tau new project` creates the config + code repos on GitHub, clones them locally, and selects the project on the active cloud.

```bash
tau --defaults --yes new project <project_name> \
  --location <local_dir> \
  --private \
  --no-embed-token
```

Resulting layout on disk:

```text
<local_dir>/<project_name>/
├── config/   # cloned from <org>/tb_<project_name>
└── code/     # cloned from <org>/tb_code_<project_name>
```

Concrete example (snake_case):

```bash
tau --defaults --yes new project my_project \
  --location "$HOME/projects" \
  --private \
  --no-embed-token
```

After creation, **immediately re-select and verify** (CLI selection is global per profile, not per repo folder):

```bash
tau --defaults --yes select project --name <project_name>
tau --defaults --yes json current
```

`Project` in the JSON output must equal `<project_name>` exactly. If it doesn't, see [selecting-taubyte-context](../selecting-taubyte-context/SKILL.md).

## B — Import an existing project pair

`tau import project` registers an existing GitHub config/code repo pair into the **currently selected cloud**. It expects **repo fullnames or ids**, not local filesystem paths.

```bash
tau --defaults --yes import project <project_name> \
  --config <org>/<config_repo_fullname> \
  --code <org>/<code_repo_fullname>
```

Concrete example:

```bash
tau --defaults --yes import project my_project \
  --config <org>/tb_my_project \
  --code   <org>/tb_code_my_project
```

Notes:

- The positional `<project_name>` is mostly a label; the cloud derives the canonical project name from the repos.
- After import, `tau` selects the imported project automatically, but still verify with `tau --json current`.

### Dream import — align `config/config.yaml` `id:`

On **Dream** (and any cloud where the project record already existed), the cloud’s canonical **project id** may **not** match the `id:` at the top of a freshly cloned **`config/config.yaml`**. Dream config-compile / inject jobs can then fail with **project ids not equal**.

After `tau import project ...` (or before first `dream inject` on that cloud), set **`config/config.yaml`** root **`id:`** to the value from:

```bash
tau --defaults --yes query project <project_name> --json
```

Then push config (`tau push project --config-only -m "..."`) before relying on builds.

## C — Clone an already-imported project locally

When the project already exists on the selected cloud and you just want a local checkout:

```bash
tau --defaults --yes clone project \
  --location <local_dir> \
  --no-embed-token
```

Result on disk:

```text
<local_dir>/<project_name>/
├── config/
└── code/
```

If you get `project not found`, the selected cloud doesn't know the project — switch clouds or import it first.

## D — Validate the project's config

```bash
tau --defaults --yes validate config -b main
```

Expected output: `Config is valid`.

If it errors at `/config.yaml:<line>:<col>: invalid variable name`, the most common cause is dashes in the project name; recreate with snake_case.

## Workflow checklist

```
New-project progress:
- [ ] Auth verified (tau --json current shows expected cloud + profile)
- [ ] Cloud selected (universe for Dream, fqdn for remote)
- [ ] Run tau new project <snake_case_name> ...
- [ ] tau select project --name <same name>
- [ ] tau --json current shows Project = <same name>
- [ ] tau validate config -b main → "Config is valid"
- [ ] Set notification.email in config/config.yaml (git config user.email is a good default)
```

## Gotchas

- **Selection drift after `new project`**: even though `tau new project` prints `Selected project:`, the global profile selection can still point at a previous project. Always re-run `tau select project --name <same>` and verify before any `tau import`/`tau push`/`tau new` resource.
- **Dashes break validation.** Re-emphasized because every other failure mode is downstream of this.
- **Empty `notification.email`.** `config/config.yaml` should have a real email under `notification.email`; an empty value can fail the config-compiler later with `mail: no address`. Prefer `git config user.email`.
- **`tau pull project` may print `already up-to-date` as an error**. This is a CLI-output quirk; in git terms it's success. Don't chase it.

## Related skills

- `authenticating-taubyte-cli`
- `selecting-taubyte-context`
- `creating-taubyte-resources`
- `pushing-taubyte-projects`
