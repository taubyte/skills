---
name: deploying-to-remote-clouds
description: Deploys to a real (non-Dream) Taubyte cloud — selects the cloud by FQDN with `tau select cloud --fqdn <fqdn>`, pushes via `tau push project` (relying on GitHub webhook delivery, no `dream inject`), and verifies with `tau query builds`, `tau query build --jid`, and `tau query logs --jid`. Encodes the project-id mismatch recovery (`project ids not equal`), the generated-domain rule (only generate while `Cloud Type: remote`), the GitHub webhook-delivery sanity check when the build table is empty, and TLS/cert expectations. Use when targeting a remote Taubyte cloud rather than Dream local, or when a deploy seems "silent" on a remote cloud.
---

# Deploying To Remote Clouds

## When to use

- Target is a real Taubyte cloud (not Dream local)
- Switching from Dream to remote (or vice versa)
- After a `tau push project` to remote, build status is unclear
- Error: `project ids not equal <patrick> != <yaml>`
- Generated domain looks wrong (or you're about to create one and aren't sure if you're on remote)
- Verifying that GitHub webhook delivery actually reached the cloud

## Mental model

A remote Taubyte cloud is reached by **FQDN** rather than by Dream universe. After `git push` (or `tau push project`), the **cloud is notified by a GitHub webhook**, builds happen automatically, and serving picks up the latest successful build. There is **no `dream inject` step** for remote — that's Dream-only. Verification therefore relies on `tau query builds/build/logs` and, when those are silent, GitHub's webhook delivery records.

## Selecting a remote cloud

```bash
tau --defaults --yes select cloud --fqdn <cloud_fqdn>
tau --defaults --yes json current
```

Verify in the JSON output:

| Field | Expected |
| --- | --- |
| `Cloud` | `<cloud_fqdn>` |
| `Cloud Type` | `remote` |
| `Profile` | The authenticated profile that has access to this cloud |

If `Cloud Type` is anything other than `remote`, **stop** before any deploy or domain create — see [selecting-taubyte-context](../selecting-taubyte-context/SKILL.md). Projects are per-cloud, so re-list:

```bash
tau --defaults --yes list projects
tau --defaults --yes select project --name <project_name>
```

## Project-id alignment (critical for remote)

Each project on a remote cloud has a canonical id stored cloud-side ("Patrick"). The local `config/config.yaml` carries the same id as `id:`. They **must match**, or every push will fail with:

```
project ids not equal <patrick_id> != <yaml_id>
```

Recovery:

```bash
# Cloud-side id (canonical)
tau --defaults --yes query project <project_name> --json | grep -i '"id"'

# Local id
awk '/^id:/{print $2; exit}' config/config.yaml
```

If they differ, **edit `config/config.yaml`** to use the cloud's canonical id, then push again. (This is one of the few cases where touching `config/config.yaml` directly is correct — it's the project identity, not a resource definition.)

## Push flow (remote)

```
Remote push progress:
- [ ] tau --json current shows Cloud Type: remote and the right Project
- [ ] Project ids match (config.yaml id == cloud canonical id)
- [ ] tau push project --config-only -m "<msg>"  (when config changed)
- [ ] Wait for config build success on the cloud
- [ ] tau push project --code-only -m "<msg>"   (when code changed)
- [ ] tau push website <site> -m "<msg>"        (when site changed)
- [ ] tau push library <lib>  -m "<msg>"        (when library changed)
- [ ] Verify each build via tau query builds / tau query logs
```

`--message` is required when `--defaults` is set — see [pushing-taubyte-projects](../pushing-taubyte-projects/SKILL.md).

**No `dream inject`** for remote. The webhook does the equivalent automatically.

## Verification commands

```bash
# Recent jobs (filter by time)
tau --defaults --yes query builds --since 1h        # default --since 168h (one week)

# Inspect a single job
tau --defaults --yes query build --jid <job_id>

# Download logs to a directory (or print to terminal if --output is omitted)
tau --defaults --yes query logs --jid <job_id> --output ./logs_<job_id>
```

`tau query builds` (alias `jobs`) returns recent builds; `tau query build` / `tau query job` (singular, `--jid` required) returns a single job; `tau query logs --jid` fetches logs for that job.

Per-resource queries (handy for verifying that the cloud sees what you just pushed):

```bash
tau --defaults --yes query project   <project_name> --json
tau --defaults --yes query function  <fn_name>      --json
tau --defaults --yes query website   <site_name>    --json
tau --defaults --yes query library   <lib_name>     --json
tau --defaults --yes query domain    <domain_name>  --json
tau --defaults --yes query database  <db_name>      --json
tau --defaults --yes query storage   <store_name>   --json
tau --defaults --yes query messaging <msg_name>     --json
tau --defaults --yes query service   <svc_name>     --json
```

## When `tau query builds` is empty after push

This usually means the cloud didn't pick up the push. The webhook is the prime suspect.

```
Empty-builds diagnosis progress:
- [ ] Confirm the git remote actually received the push (git log origin/main -1)
- [ ] On GitHub: repo → Settings → Webhooks → click the cloud webhook → Recent Deliveries
- [ ] Check the most recent delivery: status code (200/201 expected), response body, redelivery option
- [ ] If delivery failed: redeliver from GitHub UI
- [ ] If delivery succeeded but tau query builds is still empty: re-check tau --json current cloud + project; you may be querying the wrong cloud
- [ ] Cross-check via tau --defaults --yes query project <name> --json
```

GitHub's "Recent Deliveries" panel shows the actual webhook payload and the cloud's HTTP response — this is the source of truth when "did the cloud see this push?" is the question.

### Remote website/library gotchas (common “silent deploy” causes)

- **Repo not registered on that cloud**: run `tau import website <site>` / `tau import library <lib>` once on the selected remote cloud.
- **Duplicate GitHub webhooks** (from repeated imports): delete repo hooks and re-import once to recreate a single clean hook.
- **Strict website routing**: for root hosting ensure website `paths` includes exactly `"/"` (route matching is strict).
- **Builder config typos**: `.taubyte/config.yaml` must use `environment:` (not `enviroment:`); typos can make builds look “ignored”.

### If `tau query builds` is empty but you’re sure webhooks delivered

`tau query builds` can be incomplete or misleading on some remote clouds. In that case:

- fetch the **job id** from the cloud’s jobs service (Patrick), then
- pull logs directly with `tau query logs --jid <jobid>`.

Treat **job logs** as the deploy truth source, not “push succeeded”.

## Generated domains rule (remote vs Dream)

`tau new domain --generated-fqdn` produces a hostname under the **currently selected cloud's** generated suffix. On Dream the suffix is `.<universe>.localtau`. On a remote cloud it's that cloud's configured generated TLD.

**Rule**: only run `tau new domain --generated-fqdn` while `tau --json current` shows `Cloud Type: remote` (or the correct Dream universe). If you generate a domain while pointed at the wrong cloud, the FQDN won't resolve in the cloud you actually deploy to.

**Forbidden flag** (re-emphasized): never pass `--generated-fqdn-prefix` / `--g-prefix` to `tau new domain`. Use `--generated-fqdn` alone, or `--fqdn <controlled_hostname>` with proper DNS delegation.

For a custom FQDN on remote:

```bash
tau --defaults --yes new domain <name> \
  --fqdn api.example.com \
  --type auto \
  --description "<desc>"
```

`--type auto` lets the cloud handle TLS issuance.

## TLS / certificates

- Remote clouds **issue TLS certificates automatically** for generated domains under the cloud's TLD, and for custom domains where DNS delegation + verification is satisfied.
- Use `--type https` for HTTP API functions on remote (see [authoring-taubyte-function-types](../authoring-taubyte-function-types/SKILL.md)).
- A `--type http` function on a remote cloud is usually unreachable (the gateway expects TLS).

## Hosts file is **not** typically needed

Unlike Dream's `*.localtau` (which has no public DNS), remote-cloud FQDNs resolve via real DNS — generated domains under the cloud's TLD, or your custom DNS-delegated domain. **Don't add hosts entries** for remote cloud testing. If a remote URL doesn't resolve, the issue is DNS (or the domain isn't created yet), not hosts.

For Dream-specific hosts mapping, see [verifying-taubyte-functions](../verifying-taubyte-functions/SKILL.md).

## Smoke-testing a deployed function

```bash
curl -i "https://<fqdn>/<path>"
# expect HTTP 200 and the expected body
```

If the request hangs or 404s but `tau query function <fn>` shows the function exists:

- The build may not have promoted yet — wait a few seconds and re-curl.
- The domain may not be attached to the function — re-check the function YAML's `domains:` array.
- For custom domains: DNS may not have propagated; `dig <fqdn>` to check.

## Recovery — symptom → likely cause

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| `unknown cloud` on push | Cloud selection lost between sessions | `tau select cloud --fqdn <fqdn>` then retry push |
| `project ids not equal <a> != <b>` | `config/config.yaml id:` differs from cloud canonical | Update `id:` to the cloud's canonical id, push config |
| `tau query builds` empty after push | Webhook delivery failed / pointed at wrong cloud | Check GitHub Webhooks → Recent Deliveries; verify `tau --json current` |
| Generated domain FQDN not under expected TLD | Generated while pointed at Dream | Delete the domain, switch to the right cloud, recreate |
| 404 from custom domain | DNS not delegated yet, or domain `--type` issue | `dig <fqdn>`; consider `--type auto` |
| TLS error on a remote function URL | Function `trigger.type: http` (not `https`) | Switch the function to `https` (or recreate) |
| Build succeeds but old version served | Browser cache / CDN edge cache | Hard refresh; verify with `curl` first |

## Gotchas

- **`dream inject` does not work on remote.** It only triggers Dream-local builds. On remote the webhook is the trigger, period.
- **`--message` is mandatory** for `tau push` when `--defaults` is set; same rule as Dream.
- **Project selection is per-cloud.** After switching clouds, re-select the project before any push or query.
- **Don't generate domains while pointed at Dream and expect them to work on remote** — the generated TLD is per-cloud.
- **`config/config.yaml` `id:` is the project identity.** Treat it like a primary key — only edit it to align with the cloud's canonical id, never to "rename" a project.
- **GitHub webhook delivery is the truth** when build status seems silent. Don't keep retrying `tau push` if the webhook isn't being delivered; fix the webhook first.

## Related skills

- `selecting-taubyte-context` — `--fqdn` cloud selection + `tau --json current` verification
- `pushing-taubyte-projects` — push variants and the `--message` rule
- `creating-taubyte-resources` — `tau new domain` patterns (generated vs `--fqdn`)
- `authoring-taubyte-function-types` — `--type https` for remote functions
- `diagnosing-dream-builds` — analogous, but Dream-side; many concepts mirror this skill
- `understanding-taubyte-architecture` — webhook vs inject build flow
