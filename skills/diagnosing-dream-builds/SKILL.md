---
name: diagnosing-dream-builds
description: Diagnoses Dream local-cloud builds when `tau list/query builds` is empty or unreliable, by hitting the jobs HTTP endpoint directly (`GET /jobs/<project_id>`, `GET /job/<job_id>`) using the GitHub token from `~/tau.yaml`, then downloading logs with `tau query logs --jid`. Use when Dream builds appear silent, the build table is empty after `dream inject`, or you need raw job ids and logs for a failing build.
---

# Diagnosing Dream Builds

## When to use

- Just ran `dream inject push-all` / `push-specific`, but `tau query builds --since 1h` shows nothing.
- Need raw job ids that `tau` isn't surfacing.
- A build looks failed but `tau` won't print its logs.
- Verifying that a `push-specific` actually enqueued a job before chasing other failure causes.

## Mental model

`tau list/query builds` is a convenience view. On Dream local clouds it isn't always wired to local jobs. The **source of truth for Dream jobs is the local jobs HTTP endpoint** exposed by Dream's auth/seer service. Once you have a job id from there, `tau query logs --jid <jid>` can download the logs.

## Workflow

```
Progress:
- [ ] Step 1: Get the project id from config/config.yaml
- [ ] Step 2: Discover the jobs endpoint port
- [ ] Step 3: GET /jobs/<project_id> for the JobIds[] list
- [ ] Step 4: GET /job/<job_id> for an individual job object
- [ ] Step 5: tau query logs --jid <job_id> to fetch logs
```

## Step 1: Project id

```bash
awk '/^id:/{print $2; exit}' config/config.yaml
```

The project id is the value of `id:` at the root of `config/config.yaml`. You can also get it from `tau query project <name> --json`.

## Step 2: Jobs endpoint port

```bash
dream status universe <universe>
```

Find the auth/seer service port that exposes `/jobs/...`. **Ports are per-run** — rediscover every session.

Optional CORS preflight (mirrors what the Console does):

```bash
curl -i -X OPTIONS "http://localhost:<jobs_port>/jobs/<project_id>" \
  -H "Access-Control-Request-Method: GET" \
  -H "Access-Control-Request-Headers: authorization" \
  -H "Origin: https://console.taubyte.com"
```

## Step 3: List job ids for the project

```bash
TOKEN=$(awk '$1=="token:"{print $2; exit}' "$HOME/tau.yaml")
curl -sS "http://localhost:<jobs_port>/jobs/<project_id>" \
  -H "Accept: application/json" \
  -H "Origin: https://console.taubyte.com" \
  -H "Authorization: github $TOKEN"
```

Expected JSON shape:

```json
{ "JobIds": ["<jid1>", "<jid2>", ...] }
```

If the array is empty, the inject didn't enqueue a build — go back to [triggering-dream-builds](../triggering-dream-builds/SKILL.md) and confirm bootstrap (`push-all`) and any required website/library [registration](../registering-dream-repositories/SKILL.md).

## Step 4: Inspect a single job

```bash
TOKEN=$(awk '$1=="token:"{print $2; exit}' "$HOME/tau.yaml")
curl -sS "http://localhost:<jobs_port>/job/<job_id>" \
  -H "Authorization: github $TOKEN"
```

Useful fields in the response:

- `meta.repository` — which repo triggered the build (helps spot wrong-repo injects).
- `Logs` — a map of CIDs keyed by timestamp / job id; usually consumed via `tau query logs` rather than directly.
- Status fields — confirm whether the job succeeded, failed, or is still running.

## Step 5: Download logs via `tau`

```bash
tau --defaults --yes query logs --jid <job_id> --output "$HOME/projects/logs_<job_id>"
```

This writes log files into the output directory. Read them like any text file.

## Reusable helper snippet

```bash
dream_job_ids () {
  local jobs_port="$1"
  local project_id="$2"
  local token
  token=$(awk '$1=="token:"{print $2; exit}' "$HOME/tau.yaml")
  curl -sS "http://localhost:${jobs_port}/jobs/${project_id}" \
    -H "Accept: application/json" \
    -H "Authorization: github ${token}" \
    | python3 -c 'import sys,json; print("\n".join(json.load(sys.stdin).get("JobIds",[])))'
}
```

## Decision tree: build is "missing"

```
tau query builds --since 1h
  └── empty → curl /jobs/<project_id>
       ├── empty JobIds → inject didn't fire
       │     └── re-run dream inject push-all (bootstrap) — see triggering-dream-builds
       │     └── for website/library: registering-dream-repositories first
       └── non-empty → pick newest jid → tau query logs --jid <jid>
             ├── log shows compile/build error → fix code/config and push again
             └── no logs / cancelled → re-run inject and re-check
```

## Gotchas

- **Don't trust `dream inject` exit codes.** They can be `0` even when the inject didn't enqueue anything. The jobs endpoint is the truth.
- **Dynamic ports.** The jobs endpoint port changes per Dream run; always rediscover via `dream status universe <u>`.
- **Token from `~/tau.yaml`.** Use `awk '$1=="token:"{print $2; exit}'` instead of pasting the token into shell history.
- **`tau query build`** (singular) and `tau cancel build` / `tau retry build` may print `No help topic for ...` even when `tau query job` works in the same session — don't rely on those subcommands; use `tau query job --jid` and the HTTP fallback.

## Related skills

- `triggering-dream-builds`
- `registering-dream-repositories`
- `inspecting-dream-status` — find the jobs port
- `authenticating-taubyte-cli` — where the token comes from
