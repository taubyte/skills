---
name: taubyte-core-constraints
description: Non-negotiable Taubyte constraints that prevent config/build/runtime failures.
---

# Core Constraints

## Must-follow rules

1. Never ask user to run setup commands manually; agent performs setup.
2. Run CLI preflight (`taubyte-cli-prereqs`) before all Taubyte operations.
3. For Dream/local, never run backend-contacting `tau` commands before Dream is up and universe exists.
4. Use `dream` commands directly; never use `tau dream`.
5. Use default universe `default` unless user explicitly requests another.
6. After resource creation, push config (`tau push project --config-only`) before relying on resource visibility.
7. After code changes, push code (`tau push project --code-only`) and verify builds/logs.
8. Never set function timeout to `0`; use valid durations (`30s`, `1m`, etc.).
9. HTTP functions: one function per path+method; never comma-separated methods.
10. For functions/websites in automation, use empty template; for Go functions, include `--language Go`.
11. Use matcher values for DB/storage SDK calls, not resource names.
12. Do not bypass failing `tau push project` with direct git operations.
13. On Git Bash Windows for path-like flags, prefix with `MSYS_NO_PATHCONV=1`.
14. Website `.taubyte/build.sh` must be non-empty and must write deploy output to `/out`.
15. Verify project selection with `tau --json current` before resource mutations.
16. Website/library git fallback is an exception path only when `tau push website|library` is unavailable; never apply this exception to project config/code pushes.
17. Never create website/library without explicit repository strategy:
    - use `--generate-repository` with deterministic `--repository-name`, or
    - use explicit existing `--repository-name`/`--repository-id`.
18. After creating website/library, verify repository binding in config (`websites/*.yaml` or `libraries/*.yaml`) before pushing.

## Layout constraints

- Project has `tb_config_<name>` + `tb_code_<name>`.
- Website/library have separate repositories.
- Function build layout must be discoverable by build system (avoid misplaced function paths).
