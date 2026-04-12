# How to Check Logs and Debug Failed Builds

Use this workflow when a push fails or you need to inspect why a build (e.g. function compile) failed.

## 1. Check build status

From the project root (with project selected):

```bash
tau query builds
tau query builds --since 24h
```

- **Status 2** = Success
- **Status 1** = Failed → get logs (step 2)
- **Status 0** = Pending (wait and re-run)

## 2. Get the job or resource ID

You need an ID to fetch logs:

- **From CLI output:** Some flows print a resource ID (e.g. `QmU4P1Mw3Bqpgq7AXDZpXyb3rsvoJKi1C4iWhso1w9sZet`) or job ID in the error or in `tau query builds` output.
- **From error message:** e.g. "Running job `QmU4P1Mw3Bqpgq7AXDZpXyb3rsvoJKi1C4iWhso1w9sZet` failed" → use that ID.
- **From builds list:** Run `tau query builds --since 24h` and use the ID shown for the failed job.

## 3. Fetch full logs

Use the ID (positional or `--jid`):

```bash
tau query logs <resource-or-job-id>
# or
tau query logs --jid <job-id>
```

Optional: save to a directory for inspection:

```bash
tau query logs <resource-or-job-id> --output ./logs
tau query logs --jid <job-id> --output ./logs
```

## 4. What to look for in the logs

- **Per-resource blocks:** Logs are grouped by resource (e.g. timestamp or CID). Each block shows:
  - Git clone, image pull, container run
  - Per-function steps: "Building &lt;function_name&gt;", then compile/optimize output
- **Compile errors:** e.g. `package lib not found`, `exit status 1`, or Go errors (assignment mismatch, undefined method).
- **TinyGo optimizer crash:** `fatal error: unexpected signal ... SIGSEGV` in `tinygo.org/x/go-llvm` → see `build-test-functions-docker.md` (TinyGo optimizer crash) and `function-commands.md` (Remote build SIGSEGV).
- **CI/CD Errors (end of log):** Section like "CI/CD Errors:" with the high-level failure (e.g. "compressing build files failed", "artifact.wasm: The system cannot find the file specified"). That often follows an earlier compile/crash in another function.

## 5. Fix and re-run

- Fix the reported error in the **failing function** (the one named in the "Building &lt;name&gt;" step before the error).
- For compile/SDK issues: see `sdk-api-must.md` and `function-commands.md` (Troubleshooting).
- For local verification before push: see `build-test-functions-docker.md`.
- Push again and re-check: `tau push project --code-only` then `tau query builds` and `tau query logs <id>` if needed.

## Quick reference

| Step               | Command                                                                  |
| ------------------ | ------------------------------------------------------------------------ |
| List recent builds | `tau query builds` / `tau query builds --since 24h`                      |
| Get full logs      | `tau query logs <resource-or-job-id>` or `tau query logs --jid <job-id>` |
| Save logs          | `tau query logs <id> --output ./logs`                                    |
| Interpret status   | 0 = pending, 1 = failed, 2 = success                                     |

## See also

- `builds-logs-commands.md` — Full `tau query builds` / `tau query logs` syntax and flags.
- `function-commands.md` — Troubleshooting (SIGSEGV, package lib not found, SDK errors).
- `build-test-functions-docker.md` — Local Docker build to catch errors before push.
