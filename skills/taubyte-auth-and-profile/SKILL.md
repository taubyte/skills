---
name: taubyte-auth-and-profile
description: Handles Taubyte login/profile setup and first-time browser GitHub auth via tau login. Must run before cloud/project/resource operations.
---

# Auth And Profile

## First-time GitHub login (required once per machine / profile)

Before creating projects, pushing, or using generated GitHub repositories, the user must be authenticated with GitHub through the Tau CLI.

1. Check context:

   ```bash
   tau --json current
   ```

2. If **Profile** is missing, empty, or the user has not completed GitHub OAuth on this machine, **ask the user** to run in their terminal:

   ```bash
   tau login
   ```

   This opens or instructs a **browser-based GitHub sign-in**. The user must **finish the flow in the browser** (authorize the Taubyte CLI / app as prompted). The agent cannot complete this step for them.

3. After they confirm login, verify:

   ```bash
   tau --json current
   ```

## Named profile (optional)

If profile creation is needed:

```bash
tau login --new --name <profile_name> --provider github
```

If profile switch is needed:

```bash
tau login <profile_name>
```

## Verify

```bash
tau --json current
```

## Hard stop

- If GitHub-backed operations are required and the user will not run **`tau login`** to completion, stop and explain that clone, `tau new project`, `tau push`, and repo registration will fail without auth.
