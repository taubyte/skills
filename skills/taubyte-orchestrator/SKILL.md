---
name: taubyte-orchestrator
description: Master Taubyte workflow skill. Enforces strict order with Dream-by-default routing, scope routing, context logging, and verification.
---

# Taubyte Orchestrator

## When to Use

Use for any Taubyte request before running resource-specific operations.

## Mandatory gate order

1. `taubyte-auth-and-profile`
2. `taubyte-execution-modes`
3. `taubyte-core-constraints`
4. `taubyte-cloud-selection`
5. `taubyte-scope-routing`
6. `taubyte-project-and-application`
7. `taubyte-context-log`
8. `taubyte-resource-creation`
9. `taubyte-go-sdk-constraints` (for Go function code paths)
10. `taubyte-build-runtime-config` (if server-side vars/config/build logic is needed)
11. `taubyte-push-build-verify`
12. `taubyte-dream-local-operations` or `taubyte-remote-cloud-operations`
13. `taubyte-debugging-and-recovery` on failure

## Rules

- Do not skip or reorder gates.
- Default to non-interactive mode.
- Use JSON verification (`tau --json current`) after context changes.
- Keep commands sequential when selection/context can change.
- Load `taubyte-reference-index` when additional command/topic depth is needed.
- Load `taubyte-spore-drive-sdk` only for cloud deployment/self-hosting requests.
