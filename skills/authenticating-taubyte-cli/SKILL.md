---
name: authenticating-taubyte-cli
description: Authenticates the `tau` CLI to GitHub — token-based login (preferred for automation), browser OAuth fallback (`tau login --new`), and recovery from `GET /me 401`. Also explains how to retrieve the GitHub token stored in `~/tau.yaml` for direct HTTP calls. Use on first-time setup, when `tau` returns 401 Unauthorized, before any `tau new project` / `tau import` / `tau push`, or when a script needs the token to call Dream HTTP endpoints.
---

# Authenticating Taubyte CLI

## When to use

- First-time `tau` setup on a machine
- `tau` returns `GET -- \`/me\` failed with status: 401 Unauthorized`
- `tau new project` or `tau import` fails citing GitHub auth
- Need to retrieve the GitHub token for direct HTTP calls (jobs API, repository registration)

## Mental model

`tau` stores profiles in `~/tau.yaml`. Each profile holds a GitHub token used to call both the cloud API and GitHub itself. There are two ways to get a token into a profile:

1. **Token-based login** (preferred for automation): supply an existing GitHub PAT.
2. **Browser OAuth** (`tau login --new`): the CLI starts a local callback server, prints a URL, and you finish authorization in the browser.

## Token-based login (preferred)

```bash
tau login <profile_name> --provider github --token <github_token> --set-default
```

- `<profile_name>`: any local label (e.g. your username).
- `<github_token>`: a GitHub Personal Access Token with the scopes the cloud expects.
- `--set-default`: makes this profile the active one.

Verify:

```bash
tau --defaults --yes json current
```

Expect a non-error output that includes the cloud + profile.

## Browser OAuth fallback (`tau login --new`)

Use this when no usable token exists yet (e.g. truly first-time setup):

```bash
tau login --new <profile_name> --provider github
```

What happens:

- `tau` prints a `console.taubyte.com/oauth/...` URL.
- `tau` starts a local callback server on `http://127.0.0.1:<port>`.
- Open the URL in a browser, authorize, and let the redirect complete.

Caveats observed:

- In strict non-interactive environments this can fail with `cannot prompt: non-interactive mode`. Run it interactively the first time.
- If the flow ends with `no session provided`, rerun the command and complete the browser redirect again.

## Recovering from `/me 401`

Symptom:

```
GET -- `/me` failed with status: 401 Unauthorized
```

Recovery options (in order of preference):

1. **Refresh the token** in `~/tau.yaml` directly under `profiles.<name>.token`, then re-login:
   ```bash
   tau login <profile_name> --provider github --token <new_token> --set-default
   ```
2. **Re-run OAuth** if no token is available:
   ```bash
   tau login --new <profile_name> --provider github
   ```
3. **Sanity-check**:
   ```bash
   tau --defaults --yes json current
   ```

## Retrieving the token for HTTP calls

Several Dream-side flows need the same GitHub token (jobs API, repository registration). Read it from `~/tau.yaml` rather than pasting it into shell history:

```bash
TOKEN=$(awk '$1=="token:"{print $2; exit}' "$HOME/tau.yaml")
```

Use as `Authorization: github $TOKEN` in curl calls — see [diagnosing-dream-builds](../diagnosing-dream-builds/SKILL.md) and [registering-dream-repositories](../registering-dream-repositories/SKILL.md).

`~/tau.yaml` shape (simplified):

```yaml
profiles:
  <name>:
    provider: github
    token: <token>
default: <name>
```

## Preconditions for downstream tau commands

`tau new project`, `tau import project`, and several `tau` cloud queries call `GET /me` against GitHub. If auth is broken, all of them fail. **Run `tau --defaults --yes json current` after any auth change** before continuing.

## Gotchas

- **Token in shell history is a leak.** Prefer loading from `~/tau.yaml` (`awk` snippet above) when scripting.
- **Multiple profiles.** Without `--set-default`, a fresh login may not become the active profile; later `tau` commands then use a stale one.
- **OAuth in non-TTY shells fails.** If you must script, use token-based login.
- **`/me 401` cascades.** A single bad token surfaces as an opaque error in `tau new project`, `tau import`, and even some `tau push` flows. Always check `/me` first.

## Related skills

- `bootstrapping-taubyte-projects` — needs auth before any project op
- `selecting-taubyte-context` — operates on the active profile
- `diagnosing-dream-builds` and `registering-dream-repositories` — consume the token via `awk`
