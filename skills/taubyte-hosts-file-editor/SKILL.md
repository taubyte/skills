---
name: taubyte-hosts-file-editor
description: Safely add/update local hosts mappings for Dream/localtau domains so resources can be accessed by hostname.
---

# Hosts File Editor

## When to use

Use this skill when:
- the user wants to open Dream resources by hostname in browser
- local `*.localtau` domains need to resolve on the current machine
- curl tests should avoid manual `Host` headers

## Inputs

- One or more FQDNs (for example `abc123.default.localtau`)
- Target IP (default `127.0.0.1` for Dream local)
- OS (`windows`, `linux`, or `macos`)

## Hosts file paths

- Windows: `C:\Windows\System32\drivers\etc\hosts`
- Linux/macOS: `/etc/hosts`

## Procedure

1. Validate each FQDN before writing:
   - non-empty
   - no scheme/path/port
   - no duplicates
2. Read current hosts file.
3. Ensure a section marker exists:
   - `# Taubyte Local Domains`
4. For each FQDN, ensure exactly one mapping line:
   - `127.0.0.1 <fqdn>`
5. If an entry exists with a different IP, update it to target IP.
6. Preserve unrelated lines and comments.
7. Write changes with elevated permissions.
8. If elevation fails, provide exact manual steps and content to append.

## Safety rules

- Never remove unrelated host mappings.
- Never add URL prefixes such as `http://` or `https://`.
- Never include ports/paths in hosts file lines.
- Prefer idempotent updates (re-running should not duplicate entries).

## Verification

Run at least one check after edits:

```bash
curl "http://<fqdn>:<port>/"
```

Or verify direct routing without hosts dependency:

```bash
curl --resolve "<fqdn>:<port>:127.0.0.1" "http://<fqdn>:<port>/"
```

If using browser, open:

```text
http://<fqdn>:<port>/
```

## Windows manual fallback (if no elevation)

1. Open Notepad as Administrator.
2. Open `C:\Windows\System32\drivers\etc\hosts` (All Files filter).
3. Append:
   - `# Taubyte Local Domains`
   - `127.0.0.1 <fqdn>`
4. Save.
