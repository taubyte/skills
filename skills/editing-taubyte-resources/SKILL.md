---
name: editing-taubyte-resources
description: Edits and deletes Taubyte resources via `tau edit <kind> <name> [flags]` and `tau delete <kind> <name>` (covering domain, website, library, function, application, database, storage, messaging, service, smartops). Encodes the rule that domain mutations must go through the CLI (never hand-edited YAML), the `tau edit` non-interactive gotcha (some commands prompt for required list flags like `--paths` even with `--defaults --yes`), and the post-edit push + Dream inject sequence. Use when changing a resource's settings (paths/methods/match/min-max/description/tags) or removing a resource cleanly without leaving stale references.
---

# Editing Taubyte Resources

## When to use

- "Change the path / method / domain on this function"
- "Bump the database min/max replicas"
- "Rename a resource" (delete + recreate, see below)
- "Remove a resource we no longer need"
- "Update the matcher / description / tags"
- Cleaning up after a typo'd `tau new` (e.g. wrong matcher)

## Rule: CLI-managed, not YAML-managed

**Never** hand-edit `config/...` YAML to change resources. Every edit and delete goes through `tau`:

- `tau edit <kind> <name> [flags...]` — mutate fields
- `tau delete <kind> <name>` — remove the resource and its config entry

`tau` keeps related metadata in sync (e.g. domain registrations, application scope, references in other YAML files); hand edits silently break this.

For **domains** specifically, this is a hard rule (see [understanding-taubyte-architecture](../understanding-taubyte-architecture/SKILL.md) and core constraints): domains must always be CLI-managed. Don't `vim config/domains/<x>.yaml`.

## Preconditions

- `tau --defaults --yes json current` shows the **correct** Project (and the right Application for app-scoped resources). See [selecting-taubyte-context](../selecting-taubyte-context/SKILL.md).
- Local working tree is clean enough that an upcoming `tau push` (or `git push`) won't sweep up unrelated changes.

## Generic shape

```bash
# Edit
tau --defaults --yes edit <kind> <name> [field flags...]

# Delete
tau --defaults --yes delete <kind> <name>
```

For app-scoped resources (database / storage / messaging / service / smartops), select the application first:

```bash
tau --defaults --yes select application --name <app>
tau --defaults --yes edit database <db_name> --min 1 --max 4
tau --defaults --yes clear application
```

## The `tau edit` non-interactive gotcha

Some `tau edit` commands prompt interactively for **required list-style flags** even when invoked with `--defaults --yes`. The most reliably observed case is `tau edit website`, which prompts for **Paths** unless `--paths` is passed explicitly.

Working pattern (always pass list flags explicitly, even when re-using current values):

```bash
tau --defaults --yes edit website <site_name> \
  --description "<new description>" \
  --paths /
```

**General rule for `tau edit`**: when the command starts prompting despite `--defaults --yes`, re-supply the list-typed flags explicitly (`--paths`, `--methods`, `--domains`, `--tags`).

## Per-resource quick edits

### Domain

```bash
tau --defaults --yes edit domain <name> --description "<desc>"
# To change the FQDN, the safe pattern is delete + recreate:
tau --defaults --yes delete domain <name>
tau --defaults --yes new domain    <name> --fqdn <new_fqdn> --type auto
```

**Migrating Dream → remote (DNS sanity):** `*.default.localtau` validates only in Dream‑like DNS; a **remote** config compiler checks **public** resolvers (`1.1.1.1`-class) and fails with **`TXT lookup … localtau … no such host`**. **`tau delete domain` / `tau new domain … --fqdn … --type auto`** on the **remote** cloud (CLI only), then **`tau push project --config-only`**. Also remove **unused app-scoped** domains whose **`fqdn`** still ends in **`localtau`** (**`tau select application …`**, delete, **`clear application`**) — they sit in **`config/`** and can compile-fail later even if nothing references them.

### Website

```bash
tau --defaults --yes edit website <name> \
  --description "<desc>" \
  --domains <domain_name> \
  --paths /
```

### Library

```bash
tau --defaults --yes edit library <name> \
  --description "<desc>" \
  --path /libraries/<name>
```

### Function (HTTP)

```bash
tau --defaults --yes edit function <name> \
  --method POST \
  --paths /api/v2 \
  --memory 128MB \
  --timeout 5s \
  --call NewHandler \
  --tags v2
```

If you need to change the **trigger type** (HTTP → PubSub, etc.), prefer **delete + recreate**; trigger type is fundamental enough that editing in place is fragile.

### Application

```bash
tau --defaults --yes edit application <name> --description "<desc>"
```

### Database (app-scoped)

```bash
tau --defaults --yes select application --name <app>
tau --defaults --yes edit database <db_name> \
  --match <matcher> \
  --min 1 --max 4 \
  --size 1GB
tau --defaults --yes clear application
```

### Storage (app-scoped)

```bash
tau --defaults --yes select application --name <app>
tau --defaults --yes edit storage <storage_name> \
  --match <matcher> \
  --size 1GB \
  --bucket Object \
  --public
tau --defaults --yes clear application
```

### Messaging (app-scoped)

```bash
tau --defaults --yes select application --name <app>
tau --defaults --yes edit messaging <name> \
  --match <channel_match> \
  --mqtt --ws
tau --defaults --yes clear application
```

### Service (app-scoped)

```bash
tau --defaults --yes select application --name <app>
MSYS_NO_PATHCONV=1 tau --defaults --yes edit service <name> --protocol /api
tau --defaults --yes clear application
```

### Smartops

```bash
tau --defaults --yes edit   smartops <name> --description "<desc>" --tags <tag>
tau --defaults --yes delete smartops <name>
```

## Deleting any resource

```bash
tau --defaults --yes delete <kind> <name>
```

For app-scoped resources, select the application first (same shape as edit). After deletion:

- The resource YAML is removed from the config repo.
- Any function/website that referenced it (e.g. by `match`, `domains`, `source`) will break — review and update those resources next.
- If the resource owned its own GitHub repo (website/library), the GitHub repo is **not** auto-deleted. Remove or archive it manually if desired.

## Renaming = delete + recreate

`tau` doesn't have a rename in place. To rename a resource:

```
Rename progress:
- [ ] Note all callers/references (matcher strings, domains list, source paths)
- [ ] tau delete <kind> <old_name>
- [ ] tau new    <kind> <new_name> [same flags as before]
- [ ] Update any code that referenced the old name (database.New / pubsubnode.Channel / //go:wasmimport)
- [ ] tau push project --config-only -m "rename <kind> <old>->/<new>"
- [ ] After config build success: tau push project --code-only / push library / push website as needed
```

For a function that owns the only reference to a domain, the domain itself usually doesn't need recreating — domains are independent resources.

## Post-edit/-delete workflow

```
Edit/delete progress:
- [ ] Verify selection (tau --json current, application if needed)
- [ ] Run tau edit / tau delete with explicit flags
- [ ] Inspect the resulting YAML diff in config/.../<name>.yaml
- [ ] If matcher / channel / source changed, grep the code repo for old literals
    (database.New, pubsubnode.Channel, //go:wasmimport) and update them
- [ ] tau push project --config-only -m "<message>"
- [ ] Wait for the config build to succeed (see diagnosing-dream-builds)
- [ ] If code changed too: tau push project --code-only / push website / push library
- [ ] For Dream: dream inject push-specific for any changed resource repo
```

## Recovery — symptom → likely cause

After **`tau edit website …`**, always **`tau query website <name> --json`** — don’t assume the first CLI attempt stuck; **`domains`** / **`paths`** in that JSON must match what functions use when debugging same-origin **`/api/...`** (mismatch manifests as **`query website`** still pointing at **`generated`** after you thought domains moved).

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| `tau edit website` keeps prompting for "Paths" | List flag `--paths` not passed | Re-run with `--paths /` (or whichever value) |
| Edit succeeded but cloud still serves old behavior | Not pushed / not injected | `tau push project --config-only` then `dream inject push-specific` |
| `database open failed` after edit | `match` changed in YAML but Go code still uses the old literal | Grep `database.New(` and align with the new YAML `match` |
| `tau delete` complains the resource is in use | Another resource still references it | Edit/delete the referencer first, then retry delete |
| Domain edit looks applied but FQDN is unchanged | FQDN is not editable; needs delete + recreate | Use the delete + `tau new domain --fqdn ...` pattern above |
| App-scoped edit hits the wrong app | Stale application selection | `tau clear application` then `tau select application --name <correct>` |

## Gotchas

- **`tau edit` is not always idempotent across releases.** When in doubt, prefer `tau delete` + `tau new` over chained edits to a fragile resource.
- **Never delete domains that are still referenced** by websites/functions in the same project — break the references first.
- **Hand-editing YAML "just to fix one thing" is the leading cause of broken config repos.** Use the CLI; if a real bug forces a manual edit, immediately run `tau validate config -b main` to confirm the file still parses.
- **Application bleed**: forgetting to `tau clear application` after an app-scoped edit causes the next project-level command to land in the app scope.
- **Renaming a database/messaging resource breaks every caller silently** — code stays compiled, but the matcher mismatch makes opens fail at runtime. Always grep the code repo and update literals as part of the rename.
- **Smartops edit/delete round-trip works**: confirmed pattern is `tau new smartops` → `tau edit smartops` → `tau delete smartops`. Use it as the canonical sanity-check shape if a new resource type is unfamiliar.

## Related skills

- `creating-taubyte-resources` — the create side; flag tables apply to edit too
- `selecting-taubyte-context` — verify Project/Application before mutating
- `pushing-taubyte-projects` — push config (and optionally code/website/library) after edits
- `triggering-dream-builds` — bring edits into Dream
- `diagnosing-dream-builds` — check the post-edit build outcome
- `writing-taubyte-functions` — keep Go literals (`database.New`, `pubsubnode.Channel`, `//go:wasmimport`) aligned after a matcher rename
