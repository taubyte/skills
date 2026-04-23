---
name: taubyte-orchestrator
description: Master Taubyte workflow skill. Enforces strict order with Dream-by-default routing, scope routing, context logging, and verification.
---

# Taubyte Orchestrator

## When to Use

Use for any Taubyte request before running resource-specific operations.

## Mandatory gate order

1. `taubyte-cli-prereqs` (hard gate)
2. `taubyte-auth-and-profile`
3. `taubyte-execution-modes`
4. `taubyte-core-constraints`
5. `taubyte-cloud-selection`
6. `taubyte-scope-routing`
7. `taubyte-project-and-application`
8. `taubyte-context-log`
9. `taubyte-resource-creation`
10. `taubyte-go-sdk-constraints` (for Go function code paths)
11. `taubyte-build-runtime-config` (if server-side vars/config/build logic is needed)
12. `taubyte-push-build-verify`
13. `taubyte-dream-local-operations` or `taubyte-remote-cloud-operations`
14. `taubyte-debugging-and-recovery` on failure

## Optional local host access gates (Dream)

- `taubyte-hosts-file-editor` when `.localtau` resources must open by hostname.
- `taubyte-local-host-launch` to validate/launch website or API URLs using FQDN + port.

## Auto-execution policy for local runtime checks

- When user asks to "execute locally", "open URL", "run function", or "test website/function", do this automatically after push/deploy:
  1. run `taubyte-hosts-file-editor` (attempt automatic hosts update first),
  2. run `taubyte-local-host-launch` (hostname-based curl/browser-ready URLs),
  3. return concrete URLs and curl commands.
- If hosts elevation fails, provide exact manual hosts lines and continue verification with `curl --resolve`.
- If Docker is unavailable on Dream path, stop runtime claims and report the blocker explicitly.

## Rules

- Do not skip or reorder gates.
- If `taubyte-cli-prereqs` fails to verify both binaries, stop immediately.
- Default to non-interactive mode.
- Use JSON verification (`tau --json current`) after context changes.
- Keep commands sequential when selection/context can change.
- Load `taubyte-reference-index` when additional command/topic depth is needed.
- Load `taubyte-spore-drive-sdk` only for cloud deployment/self-hosting requests.
