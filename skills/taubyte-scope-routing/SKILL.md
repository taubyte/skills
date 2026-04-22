---
name: taubyte-scope-routing
description: Routes operations between global and application scopes, avoiding unnecessary application selection.
---

# Scope Routing

## Scope matrix

- Global scope (commonly): `project`, `domain`, `website`, `library`
- Application scope (commonly): `application`, `function`, `service`, `database`, `storage`, `messaging`, `smartops`

## Rules

1. If request is global-only, do not force application selection.
2. If request includes application resources, require application scope.
3. If mixed request, do global resources first, then application resources.
4. If ambiguous, ask one short question about scope.
5. Write scope decision and reason to `taubyte-context-log`.
