---
name: triggering-dream-builds
description: Triggers builds on a Dream local cloud after a GitHub push by simulating the webhook with `dream inject`. Encodes the first-run bootstrap rule (`push-all` once before `push-specific` will work reliably) and the flag-ordering gotcha for website/library repos. Use when a Dream local cloud isn't building after `git push`, when bootstrapping a freshly imported project, or when a website/library push fails with `Required flags repository-id, repository-fullname not set`.
---

# Triggering Dream Builds

## When to use

- "Dream isn't building after I pushed to GitHub"
- "First time bringing a project into Dream"
- "Website/library push didn't trigger a build"
- `dream inject push-specific` errors with `Required flags "repository-id, repository-fullname" not set`
- After every `git push` to a project's config/code/website/library repo when targeting Dream

## Mental model

Remote clouds receive a real GitHub webhook. Dream **doesn't**, so you must simulate it locally with `dream inject`. There are two shapes:

- **`push-all`** — pushes everything the cloud knows about for a project (config + code + websites + libraries).
- **`push-specific`** — pushes a single repo (finer control; expected during normal incremental work).

## First-run bootstrap rule (critical)

On a freshly imported project, **`push-specific` is unreliable until `push-all` has run at least once**. Always bootstrap with `push-all` first; afterwards prefer `push-specific` for incremental updates.

```
First-run progress:
- [ ] Step 1: `git push` config + code repos to GitHub
- [ ] Step 1.1: Ensure the code repo branch has a HEAD commit (no empty `main`)
- [ ] Step 2: `dream inject push-all <universe>` (one-time bootstrap)
- [ ] Step 3: Verify config build succeeded (see diagnosing-dream-builds)
- [ ] Step 4: Register website/library repos (see registering-dream-repositories)
- [ ] Step 5: From now on, use `push-specific` for incremental updates
```

### HEAD-less code repo gotcha (Dream inject hard fail)

If the project's code repo (e.g. `tb_code_<project>`) is on `main` but has **no commits**, Dream inject can fail with errors like:

- `getting HEAD ... reference not found`

Fix: create a minimal initial commit in the code repo and push it (via `tau push project --code-only -m "init"`), then retry inject.

## Bootstrap command (`push-all`)

```bash
dream inject push-all \
  --project-id <project_id> \
  --branch main \
  <universe>
```

- `<project_id>` comes from `config/config.yaml` (the `id:` field) or `tau query project <name> --json`.
- `<universe>` is positional and is the **last** argument (typically `default`).
- `--branch` defaults to `main`/`master` if omitted.

### `dream inject` and `--path` (local project root)

Some `dream inject` variants accept **`--path` / `-p`** pointing at an **absolute local project root**. That makes Dream build from your **working tree**, not necessarily what you last pushed to GitHub.

- **Risk:** if the tree contains **build junk** (e.g. root-owned `lib/`, `main.go`, `artifact.zip` from a bad local `tau build`), inject can compile **worse** than a clean GitHub clone.
- **Default:** after `tau push`, run inject **without** `--path` so Dream clones from GitHub — unless you have explicitly verified the workspace is clean.

## Incremental command (`push-specific`)

Use **long flags** and put the universe **last** — this is the only ordering observed to work reliably for website/library repos:

```bash
dream inject push-specific \
  --repository-id <repo_id> \
  --repository-fullname <org>/<repo> \
  --project-id <project_id> \
  --branch main \
  <universe>
```

Where to find each value:

| Value | Source |
| --- | --- |
| `<repo_id>` | `websites/<name>.yaml` or `libraries/<name>.yaml` → `source.github.id`; or GitHub API |
| `<org>/<repo>` | The fullname (e.g. `<org>/tb_code_<project>_<site>`) |
| `<project_id>` | `config/config.yaml` → `id:` |
| `<universe>` | The universe Dream was started with (typically `default`) |

For a config-or-code-only push, the same shape works against the config repo's `repository-id`/`fullname`, or you can fall back to `push-all`.

## Validation (do not trust exit code alone)

`dream inject` can return exit code `0` while still failing. After the call:

- Scan stdout/stderr for `failed`, `error`, `inject`, or missing-resource messages.
- Confirm a build appeared via `tau query builds --since 1h` or the jobs API fallback (see [diagnosing-dream-builds](../diagnosing-dream-builds/SKILL.md)) — **`GET /jobs/<project_id>`** should return **`JobIds`** once work is really queued; do not trust exit **0** alone.
- If a build doesn't appear and the repo is a website/library, run [registering-dream-repositories](../registering-dream-repositories/SKILL.md) first, then retry.

## Common failure → recovery

| Symptom | Likely cause | Recovery |
| --- | --- | --- |
| `Required flags "repository-id, repository-fullname" not set` | Short flags or wrong order | Use the long-flag block above with universe **last** |
| `unknown cloud` from `tau` afterwards | Cloud selection drifted | `tau --defaults --yes select cloud --universe <u>` then retry |
| `push-specific` succeeds but no build appears | Repo not registered with local auth/repository service | See [registering-dream-repositories](../registering-dream-repositories/SKILL.md) |
| `tau query builds` empty | Build view not wired to Dream-local jobs | Use jobs API HTTP fallback in [diagnosing-dream-builds](../diagnosing-dream-builds/SKILL.md) |
| `project ids not equal <cloud> != <yaml>` in config job logs | `config/config.yaml` `id:` does not match the Dream project id | Set `config/config.yaml` `id:` to `tau query project <name> --json` `id`, push config, then inject again |

## Preconditions

- Dream universe is up: `dream status universe <universe>` (see [inspecting-dream-status](../inspecting-dream-status/SKILL.md)).
- Repos have been pushed to GitHub already; `dream inject` only **triggers** the cloud to fetch from GitHub.
- Project has been imported into the local cloud via `tau import project ...`.

## Related skills

- `starting-dream-locally`
- `inspecting-dream-status`
- `registering-dream-repositories`
- `diagnosing-dream-builds`
- `pushing-taubyte-projects` — the GitHub push that comes before each inject
