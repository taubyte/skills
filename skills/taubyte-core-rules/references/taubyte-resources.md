---
description: Resource creation - detection, validation, .taubyte, checklists
globs: "**/config/**/*.yaml"
alwaysApply: false
---

# Taubyte Resource Creation

## RESOURCE CREATION

### Resource Detection

Parse user request: Website ("website", "site", "web app"), Database ("database", "db", "store data", "persist"), Storage ("storage", "files", "upload"), Functions ("api", "endpoint", "function", "backend"), Messaging ("pubsub", "messaging", "notifications"), Domains (auto-create if website/functions need).

### Resource Creation Order

1. Domains (if needed)
2. Database
3. Storage
4. Messaging
5. Functions
6. Website

### After Each Resource Creation

**[MUST] Pre-Creation:** Verify `tau current` shows correct project. Select project first if not.

**[MUST] Post-Creation Validation:**

1. **Verify Config File:** `ls [project]/config/[resource-type]/[name].yaml` (or `applications/[app]/[resource-type]/[name].yaml` for applications). IF missing: DO NOT proceed - fix config first
2. **Push Config:** `tau push project --config-only --message "Add [resource-type] [name]"`
3. **Code changes:** `tau push project --code-only --message "Update [resource-type] code"`
4. **Validate exists:** `tau list [resource-type]` shows [name]. IF not: fix config before code generation

**VALIDATE:** Config exists, pushed, resource in list. ONLY proceed with code if ALL pass.

### Resource Checklists

**Domain:** Create, push config, validate. IF Dream/Local: Edit hosts file (see taubyte-dream Hosts File Protocol).

**Database/Storage/Messaging:** Create, push config, validate exists.

**Function:** Create, VERIFY `--source` and `--call` included, VERIFY `--template empty --language Go`. **[MUST]** HTTP: use ONE `--method` per function (e.g. `--method GET` only); NEVER comma-separated methods (e.g. `GET,POST,DELETE`)—config build fails. Create separate functions per path+method (e.g. get_notes, save_note, delete_note). Handle `empty.go`: Rename to `[functionName].go` if that doesn't exist; else delete `empty.go`. Verify/Create `.taubyte`. Push config, validate.

**Website:** Verify project selected. Check repo existence before `--generate-repository`. Create, IF conflict use `--repository-name` instead. Verify config exists, push, validate `tau list websites`. Clone, verify branch main, create .taubyte, push website code.

**Library:** Check repo existence. Create, IF conflict use `--repository-name`. Clone, verify branch main, .taubyte, push config, validate.

## .taubyte FOLDER REQUIREMENTS

**Common:** All have `config.yaml` and `build.sh`. Build outputs to `/out`. Verify before deploying.

### Function .taubyte

Location: `tb_code_[project]/functions/[name]/.taubyte/`

```yaml
# config.yaml
version: 1.00
environment:
  image: taubyte/go-wasi:latest
  variables:
workflow:
  - build
```

```bash
# build.sh
#!/bin/bash
. /utils/wasm.sh
build "${FILENAME}"
ret=$?
echo -n $ret > /out/ret-code
exit $ret
```

Functions use `environment:` (correct). FILENAME is auto-set.

### Library .taubyte

Location: `tb_library_[name]/.taubyte/`

```yaml
enviroment:  # Note: typo in Taubyte docs
  image: golang:1.21-alpine
```

Build: `go build -o /out/library.a .`

### Website .taubyte

Location: `tb_website_[name]/.taubyte/`

```yaml
enviroment:  # Note: typo in Taubyte docs
  image: node:16.18.0-alpine
```

Build: `yarn build && mv dist/* /out` or `npm run build && cp -r dist/* /out`

**Note:** Libraries/websites use `enviroment:` (typo). If build fails with "unknown key," try `environment:`. See `.cursor/taubyte-folder-examples.md` for full examples.
