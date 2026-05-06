---
name: taubyte-always-follow-rules
description: Non-negotiable Taubyte workflow rules for agents—use `tau` for resources, never invent layouts, choose one build strategy (inline vs library), keep scopes correct, and follow push/inject ordering. Read this FIRST before acting on any Taubyte request.
---

# Taubyte Always-Follow Rules (Agent Gate)

## When to use

Use this skill as the **first read** for any task that mentions:

- Taubyte / Tau / Dream
- `tau new` / `tau push` / `dream inject`
- domains, websites, libraries, functions, databases, storage, messaging, services

## Hard rules (do not violate)

1. **Never create resources manually.** Create/update/delete Taubyte resources only via:
   - `tau new ...`
   - `tau edit ...`
   - `tau delete ...`
   Especially: **domains are always CLI-managed** (never hand-edit `config/domains/*.yaml`).

2. **Never invent folder structure.** Code and resource files must live exactly where `tau` created/imported them.
   - Do not “make it work” by creating new top-level folders.
   - Do not move function dirs unless a Taubyte build log explicitly proves the builder expects a different path (see rule 10).

3. **Read the relevant Taubyte skills before choosing an implementation.**
   - Minimum reads before acting: `installing-taubyte-tooling`, `selecting-taubyte-context`, `creating-taubyte-resources`, `pushing-taubyte-projects`.
   - If writing Go handlers: also read `writing-taubyte-functions` and `configuring-taubyte-build-runtime`.
   - If building a website: also read `building-taubyte-websites`.
   - If Dream-local: also read `starting-dream-locally` and `triggering-dream-builds`.

4. **Only create an application when asked or required.**
   - Application-scoped resources include: database, storage, messaging, service.
   - If the user didn’t ask for persistence or app-scoped infra, don’t create an application “just in case”.

5. **Choose ONE function build strategy upfront.**
   - **Inline function**: `tau new function ... --source .`
     - Each function builds separately.
   - **Library-sourced function**: `tau new library ...` + implement handlers in the library, then
     `tau new function ... --source libraries/<lib>`
     - Library builds once; multiple functions can reuse it.
   - Do not mix both patterns for the same endpoint.

6. **Function wiring must match exactly.**
   - `execution.call` equals the Go `//export <Call>` symbol exactly.
   - HTTP handler ordering is strict: **Headers → Write → Return(status)**.

7. **Build/runtime files must be present and correct.**
   - `.taubyte/build.sh` must be **non-empty** and executable.
   - Websites must copy deployable assets into **`/out`**.
   - Do not put build-time env vars in `.taubyte/config.yaml`; export them in `build.sh`.

8. **Always verify selection before mutations.**
   - Before any `tau new/edit/delete/push`: `tau --json current` must show the expected Cloud + Project (+ Application only when needed).

9. **Push order is non-negotiable.**
   - `tau push project --config-only -m "..."` first (wait for config build success),
   - then push code/website/library as needed.

10. **Path mismatch recovery rule (only allowed case for moves).**
   - If a build log says it cannot find `.taubyte` under a specific path, treat that path as ground truth and align your on-disk layout to it.
   - Otherwise, do not move directories preemptively.

## Minimal workflow (copy/paste)

```
# Gate
tau --defaults --yes --json current

# Create resources
tau --defaults --yes new domain ...
tau --defaults --yes new website ... && tau --defaults --yes import website ...
tau --defaults --yes new library ... && tau --defaults --yes import library ...
tau --defaults --yes new function ...   # choose inline or library source

# Push
tau --defaults --yes push project --config-only --message "..."
tau --defaults --yes push project --code-only   --message "..."
tau --defaults --yes push website <name>        --message "..."
tau --defaults --yes push library <name>        --message "..."

# Dream-local (if applicable)
dream inject push-all <universe>
```

## Related skills

- `enforcing-taubyte-constraints` (auditing checklist)
- `creating-taubyte-resources` (how to create each resource)
- `editing-taubyte-resources` (how to change resources)
- `pushing-taubyte-projects` (push ordering, required message)
- `writing-taubyte-functions` (Go SDK + handler rules)
- `configuring-taubyte-build-runtime` (build.sh/config.yaml rules)
- `building-taubyte-websites` (the `/out` rule)
- `triggering-dream-builds` (Dream inject rules)

