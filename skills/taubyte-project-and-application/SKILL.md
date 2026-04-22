---
name: taubyte-project-and-application
description: Creates/selects project and conditionally selects application based on scope routing.
---

# Project And Application

## Steps

1. Verify context:
   ```bash
   tau --json current
   ```
2. Create/select project:
   ```bash
   tau --defaults --yes new project --name <project> --description "<desc>" --private --no-embed-token
   tau select project --name <project>
   ```
3. If application scope is needed:
   ```bash
   tau --defaults --yes new application --name <app> --description "<desc>"
   tau select application --name <app>
   ```
4. Verify:
   ```bash
   tau --json current
   ```
