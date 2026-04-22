---
name: taubyte-auth-and-profile
description: Handles Taubyte login/profile setup. Must run before cloud/project/resource operations.
---

# Auth And Profile

## Steps

1. Check context:
   ```bash
   tau --json current
   ```
2. If profile is missing, create one:
   ```bash
   tau login --new --name <profile_name> --provider github
   ```
3. If profile switch is needed:
   ```bash
   tau login <profile_name>
   ```
4. Verify:
   ```bash
   tau --json current
   ```
