---
name: selecting-taubyte-context
description: Keeps `tau` pointing at the right cloud and project before any mutation — `tau current` / `tau --json current`, `tau select cloud --universe|--fqdn`, `tau select project`, and the `tau clear project` + reselect recovery. Encodes the "projects are per cloud" rule and the `unknown cloud` recovery. Use before any `tau push`, `tau delete`, `dream inject`, or resource creation, and whenever `tau` appears to operate on the wrong project.
---

# Selecting Taubyte Context

## When to use

- Before any `tau push`, `tau delete`, `tau new` resource, or `dream inject` call.
- Before/after switching between Dream local and remote clouds.
- `tau` operations target a different project than the local repo on disk.
- `tau push project` fails with `unknown cloud` despite `tau current` showing one.

## Mental model

`tau` stores **profile** (auth) and **selection** (cloud + project + optional application) in the global `~/tau.yaml`, **not** in the on-disk project folder. So `cd`-ing into a project doesn't change `tau`'s selection; you must select explicitly. Projects are scoped **per cloud** — switching clouds can leave the project selection pointing at something that doesn't exist on the new cloud.

**Dream / import drift:** the **`id:`** in your cloned **`config/config.yaml`** must match the **cloud’s** project id for that name on the selected cloud (see [bootstrapping-taubyte-projects](../bootstrapping-taubyte-projects/SKILL.md) — "Dream import — align `config/config.yaml` `id:`"). If builds reference the wrong id, fix the YAML and push config before `dream inject`.

## Quick checks

```bash
tau current
tau --defaults --yes json current
```

The JSON form is preferred for scripting; it shows `Cloud`, `Cloud Type`, `Project`, and `Application` fields explicitly.

## Selecting a cloud

### Dream (local)

Dream clouds are addressed by **universe**:

```bash
tau --defaults --yes select cloud --universe default
tau --defaults --yes select cloud --universe blackhole
```

The universe must be one that's actually running — see [inspecting-dream-status](../inspecting-dream-status/SKILL.md).

### Remote

Remote clouds are addressed by **fqdn**:

```bash
tau --defaults --yes select cloud --fqdn <cloud-fqdn>
```

After selecting a cloud, **list its projects** because the previous selection may not exist there:

```bash
tau --defaults --yes list projects
tau --defaults --yes list projects --json
```

## Selecting a project

```bash
tau --defaults --yes select project --name <project_name>
tau --defaults --yes json current   # verify Project = <project_name>
```

If `Project` doesn't match `<project_name>` after a `select`, clear and reselect:

```bash
tau --defaults --yes clear project
tau --defaults --yes select project --name <project_name>
tau --defaults --yes json current
```

## Selecting / clearing an application

Application is optional and only needed for app-scoped resources (database, storage, messaging, service, app-scoped functions).

```bash
tau --defaults --yes select application --name <app_name>
# ... do app-scoped work ...
tau --defaults --yes clear application
```

Always `clear application` when returning to project-level resources, or later `tau new domain|website|library` calls can land in the wrong scope.

## Pre-mutation routine (always)

Run this block **before** any push, delete, inject, or `tau new` resource:

```bash
tau --defaults --yes json current

# If cloud is wrong:
tau --defaults --yes select cloud --universe <expected_universe>   # or --fqdn <fqdn>

# If project is wrong:
tau --defaults --yes select project --name <expected_project>

# Re-verify:
tau --defaults --yes json current
```

Treat any mismatch between the JSON `Project` and the project the user is editing on disk as a **hard stop** — fix selection before proceeding.

## Recovery — `unknown cloud` on push

`tau push project` can fail with `unknown cloud` even though `tau current` shows a cloud. Recovery:

```bash
tau --defaults --yes select cloud --universe <expected_universe>
tau --defaults --yes push project --message "<msg>"
```

Re-selecting the cloud refreshes the internal binding `tau push` needs.

## Recovery — wrong project after switching clouds

```bash
# 1. Confirm what the new cloud knows
tau --defaults --yes list projects --json

# 2. If <expected_project> is missing, you're on the wrong cloud or it must be imported.
#    Otherwise, select it:
tau --defaults --yes select project --name <expected_project>
tau --defaults --yes json current
```

## Hygiene checklist

```
Before any mutation:
- [ ] tau --json current shows expected Cloud + Cloud Type
- [ ] Project field equals the on-disk project name
- [ ] Application is either the expected one or cleared
- [ ] For Dream: dream status universe <name> works
```

## Real-world failure pattern

A push can target the wrong project when an old selection persists from a prior session — e.g. a dashed-name project (`my-project`) is still selected globally while you're editing the snake_case version (`my_project`) on disk. The `tau --json current` check catches this; always run it before `push` / `delete` / `inject`.

## Gotchas

- **Profile selection is global.** A teammate's terminal, a different session, or `tau new project` itself can leave you pointing somewhere unexpected. The only safe pattern is verify-on-each-mutation.
- **Application bleeds into project ops.** If an application is selected, `tau new domain|website|library` can interpret commands in the app scope. Clear it for project-level work.
- **Dream universes vs remote fqdns are mutually exclusive.** Picking one replaces the other.

## Related skills

- `authenticating-taubyte-cli`
- `bootstrapping-taubyte-projects`
- `creating-taubyte-resources`
- `pushing-taubyte-projects`
