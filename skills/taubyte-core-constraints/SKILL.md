---
name: taubyte-core-constraints
description: Non-negotiable Taubyte constraints that prevent config/build/runtime failures.
---

# Core Constraints

## Must-follow rules

1. Never ask user to run setup commands manually; agent performs setup.
2. For Dream/local, never run backend-contacting `tau` commands before Dream is up and universe exists.
3. Use `dream` commands directly; never use `tau dream`.
4. Use default universe `default` unless user explicitly requests another.
5. After resource creation, push config (`tau push project --config-only`) before relying on resource visibility.
6. After code changes, push code (`tau push project --code-only`) and verify builds/logs.
7. Never set function timeout to `0`; use valid durations (`30s`, `1m`, etc.).
8. HTTP functions: one function per path+method; never comma-separated methods.
9. For functions/websites in automation, use empty template; for Go functions, include `--language Go`.
10. Use matcher values for DB/storage SDK calls, not resource names.
11. Do not bypass failing `tau push project` with direct git operations.
12. On Git Bash Windows for path-like flags, prefix with `MSYS_NO_PATHCONV=1`.
13. Website `.taubyte/build.sh` must be non-empty and must write deploy output to `/out`.
14. Verify project selection with `tau --json current` before resource mutations.
15. Website/library git fallback is an exception path only when `tau push website|library` is unavailable; never apply this exception to project config/code pushes.

## Layout constraints

- Project has `tb_config_<name>` + `tb_code_<name>`.
- Website/library have separate repositories.
- Function build layout must be discoverable by build system (avoid misplaced function paths).
