---
name: registering-dream-repositories
description: Registers a website or library GitHub repository with the local Dream auth/repository service via `PUT /repository/github/<repo_id>` so hooks/build triggers are tracked. Use when, after `tau import website|library`, a `dream inject push-specific` for that repo doesn't produce a build, or when the local cloud reports the repo as unknown despite being in the project's config YAML.
---

# Registering Dream Repositories

## When to use

- A website or library was just imported (`tau import website|library`), and `dream inject push-specific` for that repo isn't producing a build.
- The project was bootstrapped with `dream inject push-all` but website/library builds are still silent.
- The local cloud doesn't seem to know about a repo even though `websites/<name>.yaml` / `libraries/<name>.yaml` references it.
- Mirroring what the Console does when wiring a GitHub repo to a local Taubyte cloud.

## Why this is needed

A remote cloud receives GitHub webhooks automatically. On Dream, the local auth/repository service has to be told about each website/library repo before it will accept hook-style pushes for it. `tau import website|library` clones the repo locally but does **not** always register it with the local service.

## Workflow

```
Progress:
- [ ] Step 1: Bootstrap the project once with `dream inject push-all`
- [ ] Step 2: Find each website/library repo's GitHub repo id
- [ ] Step 3: Register each repo via PUT /repository/github/<repo_id>
- [ ] Step 4: Verify the registration via GET
- [ ] Step 5: Now `dream inject push-specific` for that repo will work
```

**Order matters**: do the project `push-all` bootstrap first (see [triggering-dream-builds](../triggering-dream-builds/SKILL.md)), then register each website/library, then incremental `push-specific` for those repos.

## Step 1: Find the repo id

Each website/library has a `source.github.id` in its config YAML:

```bash
# website
grep -A2 'github:' config/websites/<name>.yaml
# library
grep -A2 'github:' config/applications/<app>/libraries/<name>.yaml \
  || grep -A2 'github:' config/libraries/<name>.yaml
```

Optional cross-check against GitHub:

```bash
TOKEN=$(awk '$1=="token:"{print $2; exit}' "$HOME/tau.yaml")
curl -sS "https://api.github.com/repositories/<repo_id>" \
  -H "Authorization: token $TOKEN"
```

## Step 2: Find the repository service port

The local repository service is one of the Dream services. Discover its port from:

```bash
dream status universe <universe>
```

Look for the auth/repository service entry; use its HTTP port as `<repo_port>` below.

## Step 3 (optional): CORS preflight

```bash
curl -i -X OPTIONS "http://localhost:<repo_port>/repository/github/<repo_id>" \
  -H "Access-Control-Request-Method: PUT" \
  -H "Access-Control-Request-Headers: authorization,content-type" \
  -H "Origin: https://console.taubyte.com"
```

## Step 4: Register the repo (PUT)

```bash
TOKEN=$(awk '$1=="token:"{print $2; exit}' "$HOME/tau.yaml")
curl -sS -X PUT "http://localhost:<repo_port>/repository/github/<repo_id>" \
  -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  -H "Origin: https://console.taubyte.com" \
  -H "Authorization: github $TOKEN" \
  --data-raw '{}'
```

Expected response shape:

```json
{"key":"/repositories/github/<repo_id>/key"}
```

## Step 5: Verify (GET)

```bash
TOKEN=$(awk '$1=="token:"{print $2; exit}' "$HOME/tau.yaml")
curl -sS "http://localhost:<repo_port>/repository/github/<repo_id>" \
  -H "Accept: application/json" \
  -H "Origin: https://console.taubyte.com" \
  -H "Authorization: github $TOKEN"
```

Expected shape (a non-empty `hooks` array means the repo is registered and tracked):

```json
{"hooks":["12345"]}
```

## Reusable helper snippet

```bash
register_repo () {
  local repo_id="$1"
  local repo_port="$2"
  local token
  token=$(awk '$1=="token:"{print $2; exit}' "$HOME/tau.yaml")

  curl -sS -X PUT "http://localhost:${repo_port}/repository/github/${repo_id}" \
    -H "Accept: application/json" \
    -H "Content-Type: application/json" \
    -H "Origin: https://console.taubyte.com" \
    -H "Authorization: github ${token}" \
    --data-raw '{}'
  echo
  curl -sS "http://localhost:${repo_port}/repository/github/${repo_id}" \
    -H "Accept: application/json" \
    -H "Origin: https://console.taubyte.com" \
    -H "Authorization: github ${token}"
  echo
}
```

## Gotchas

- **Order**: register the website/library repos **after** the project bootstrap (`push-all`), not before. Without bootstrap, the cloud may not yet know the project.
- **Token security**: the GitHub token comes from `~/tau.yaml`. Don't paste it into shell history; load it with `awk` as shown.
- **Empty `hooks`**: if the verify GET returns `{"hooks":[]}`, the registration didn't take. Re-check the repo id, the token, and that Dream's repository service port is right.
- **Re-running**: PUT is idempotent for the same repo id. Safe to re-register if state seems off.

## Related skills

- `triggering-dream-builds` — bootstrap and incremental injects
- `inspecting-dream-status` — find the repository service port
- `authenticating-taubyte-cli` — where the token comes from (`~/tau.yaml`)
- `diagnosing-dream-builds` — check whether builds actually fire after registration
