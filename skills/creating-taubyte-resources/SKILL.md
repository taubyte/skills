---
name: creating-taubyte-resources
description: Creates Taubyte resources non-interactively via `tau new` for domain, website, library, function, application, database, storage, messaging, and service. Encodes the project-vs-application scope rule, the database `min < max` constraint, the website/library `--generate-repository` + import sequence, and the forbidden `--generated-fqdn-prefix` flag. Use when adding any resource to a Taubyte project's config repo.
---

# Creating Taubyte Resources

## When to use

- "Add a domain / website / library / function / database / storage / messaging / service"
- "Create an application for app-scoped resources"
- Building out a project after `bootstrapping-taubyte-projects`
- Reviewing or scripting non-interactive resource creation

## Intent-first resource planning (agent behavior)

When a user asks to "create an app" (not just a resource), first infer the **minimum Taubyte resource set** required to make that app real, then create those resources with correct scope + wiring.

General heuristics:

- **Web app UI** → create a **website** (project-scoped if needed or mentioned) + a **domain** (project-scoped if mentioned) and attach the site to the domain and path(s).
- **Persistent data** → create a **database** (application-scoped if needed). create storage if needed ; choose only when implied by the intent.
- **UI ↔ data/API glue** → create at least one **HTTP function** that reads/writes the database and returns JSON for the website to consume. Attach the function to the same domain when a same-origin `/api/...` pattern is implied.

Rules while doing this:

- Prefer the **fewest resources** that satisfy the intent, but do not omit the glue (e.g. database without a function is usually not an "app").
- Keep scopes correct: database/storage/messaging/service are **application-scoped** if needed or mentioned; website/domain are **project-scoped** if needed or mentioned.
- Use match strings as stable wiring keys (database `match`, messaging channel, service protocol) and keep them consistent with Go calls and triggers (see [enforcing-taubyte-constraints](../enforcing-taubyte-constraints/SKILL.md) rule 16).

## Preconditions

- `tau --defaults --yes json current` shows the **correct** Project (and the right Application, or none, depending on scope) — see [selecting-taubyte-context](../selecting-taubyte-context/SKILL.md).
- Project has been created/imported — see [bootstrapping-taubyte-projects](../bootstrapping-taubyte-projects/SKILL.md).

## Scope rule (project vs application)

| Resource | Scope |
| --- | --- |
| `domain` | Project |
| `website` | Project |
| `library` | Project |
| `function` (HTTP) | Project (default) or Application |
| `application` | Project (creates the application) |
| `database` | **Application** |
| `storage` | **Application** |
| `messaging` | **Application** |
| `service` | **Application** |

For app-scoped resources: `tau --defaults --yes select application --name <app>` first; for project-scoped, ensure no application is selected (`tau --defaults --yes clear application`).

## Domains

### Generated FQDN (Dream / quick demos)

```bash
tau --defaults --yes new domain <domain_name> \
  --generated-fqdn \
  --type auto \
  --description "<desc>"
```

Result: a random FQDN of the form `<random>.<universe>.localtau`.

### Custom FQDN (remote, controlled hostname)

```bash
tau --defaults --yes new domain <domain_name> \
  --fqdn <fqdn> \
  --type auto \
  --description "<desc>"
```

### Forbidden flag

**Never** pass `--generated-fqdn-prefix` (or `--g-prefix`) — it's not supported by policy. Use `--generated-fqdn` alone or use `--fqdn` with a controlled hostname.

## Websites

```bash
tau --defaults --yes new website <site_name> \
  --domains <domain_name> \
  --paths / \
  --template empty \
  --generate-repository \
  --repository-name tb_code_<project>_<site_name> \
  --private \
  --no-embed-token \
  --branch main

tau --defaults --yes import website <site_name>
tau --defaults --yes list websites
```

Notes:

- `--generate-repository` creates a **separate** GitHub repo for the website; pair with a deterministic `--repository-name` (recommended: `tb_code_<project>_<site>`).
- After creation, run `import website` so the local clone appears under `<project>/websites/<repo>/`.
- Author build content under `.taubyte/build.sh` writing to `/out` — see [building-taubyte-websites](../building-taubyte-websites/SKILL.md).

To attach an existing website repo instead of generating one, use `--repository-name <fullname>` or `--repository-id <id>` and drop `--generate-repository`.

## Libraries

```bash
tau --defaults --yes new library <lib_name> \
  --path /libraries/<lib_name> \
  --template empty \
  --generate-repository \
  --repository-name tb_code_<project>_<lib_name> \
  --private \
  --no-embed-token \
  --branch main

tau --defaults --yes import library <lib_name>
tau --defaults --yes list libraries
```

`--path` is the path **inside the library repo** where sources live; it must match the actual layout in the cloned repo.

## HTTP functions

```bash
tau --defaults --yes new function <fn_name> \
  --type http \
  --method GET \
  --paths /<path> \
  --domains <domain_name> \
  --language Go \
  --source . \
  --memory 64MB \
  --timeout 30s \
  --call <ExportedHandler> \
  --template empty
```

Or use a known-good ping_pong template:

```bash
tau --defaults --yes new function ping \
  --type http --method GET --paths /ping \
  --generate-domain \
  --language Go --source . \
  --memory 64MB --timeout 1s \
  --call ping \
  --template ping_pong --use-template
```

Function rules:

- One function per (path, method) — never comma-separated methods.
- `--call` must equal a `//export <name>` symbol in the source.
- For Go, edit the function root `empty.go` (next to `go.mod`) — never create a `lib/` tree or hand-author `main.go`.

## Applications

```bash
tau --defaults --yes new application <app_name> \
  --description "<desc>"
tau --defaults --yes select application --name <app_name>
tau --defaults --yes list applications
```

## Databases (application-scoped)

```bash
tau --defaults --yes new database <db_name> \
  --match <matcher> \
  --min 1 --max 2 \
  --size 1GB
```

**Constraint observed**: `--min 1 --max 1` fails with `min(1) cannot be greater than max(1)`. Use `--min 1 --max 2` (or higher max).

The `--match` value is the **runtime key**: Go code must call `database.New("<matcher>")` with the **identical** string (including any leading `/`).

## Storage (application-scoped)

```bash
tau --defaults --yes new storage <storage_name> \
  --match <matcher> \
  --size 1GB \
  --bucket Object \
  --public
```

## Messaging (application-scoped)

```bash
tau --defaults --yes new messaging <msg_name> \
  --match <channel_match> \
  --mqtt --ws
```

The `<channel_match>` is the channel string. Producers in Go call `pubsub/node`'s `Channel("<channel_match>")`, and PubSub-triggered functions set `trigger.channel: <channel_match>`.

## Services (application-scoped)

```bash
tau --defaults --yes new service <svc_name> \
  --protocol /api
```

On Git Bash / MSYS, prefix path-like flags with `MSYS_NO_PATHCONV=1` to avoid path mangling.

## Workflow checklist for a typical project

```
Resource creation progress:
- [ ] Verify selection (tau --json current)
- [ ] Create domain (project scope)
- [ ] Create website + import (project scope)
- [ ] Create library + import (project scope)
- [ ] Create application
- [ ] Select application
- [ ] Create database / storage / messaging / service
- [ ] Create functions (HTTP at project scope, others at app scope as needed)
- [ ] tau clear application
- [ ] tau push project --config-only --message "..."
- [ ] Wait for config build success before pushing website/library code
```

## Gotchas

- **Dream vs CLI vs remote WASM paths:** `tau new function ... --source .` with an application selected may scaffold under **`code/applications/<app>/functions/<fn>/`** in your clone, while **Dream** or **remote** builders often expect **`code/<app>/functions/<fn>/`** (no `applications/` segment). A remote job may fail with **couldn't find `.taubyte`** under `.../<app>/functions/<fn>` while your tree still has `.../applications/<app>/functions/<fn>`. Treat the **path in the first failing build log** as ground truth and **move** the function directory (and fix any relative references) so `.taubyte/` sits where the builder looks — do not guess a different layout.
- **Application bleed**: forgetting `tau clear application` causes later `tau new domain|website|library` to land in the app instead of project scope.
- **Matcher = runtime string**: `match` in YAML must equal the literal Go calls (`database.New("...")`, `pubsubnode.Channel("...")`, `trigger.channel`). Mismatched leading `/` is a frequent cause of "open failed" / silent producers.
- **Repo strategy must be explicit** for websites/libraries — either `--generate-repository` with a deterministic `--repository-name`, or `--repository-name`/`--repository-id` for an existing repo.
- **Don't hand-edit domain YAML.** Always use `tau new domain` / `tau delete domain` / `tau query domain`.
- **Min/max on database**: `--min` must be strictly less than `--max`.

## Related skills

- `bootstrapping-taubyte-projects`
- `selecting-taubyte-context`
- `pushing-taubyte-projects`
- `building-taubyte-websites`
- `verifying-taubyte-functions`
