---
name: tb_cmd_tau_new_project
description: Creates a new project non-interactively with deterministic flags and follow-up verification.
---

## Command card

**Preconditions:** Auth OK; cloud selected; distinct project name.

**Command:**

```bash
tau --defaults --yes new project --name <project> --description "<desc>" --private --no-embed-token
```

## Mandatory follow-up (same agent turn — do not skip)

`tau` stores **Project** in the **global profile**, not in the repo folder. `tau new project` may print `Selected project: <project>`, but you **must** still run **`tau select project`** immediately after, then verify. Otherwise the next `tau json current` (or another chat / later step) can still show an **old** project (e.g. a previous `skills-e2e-local` selection).

**Always run this block next, with the same `<project>` name:**

```bash
tau --defaults --yes select project --name <project>
tau --defaults --yes json current
```

**Hard stop:** If **Project** in that output is not **exactly** `<project>`, fix cloud/profile/selection before **any** `tau import`, `tau push project`, `tau new` resource, or Dream inject.

**Postconditions:** `tb_cmd_tau_select_project` satisfied; never import/push until `tb_cmd_tau_json_current` verifies **Project** equals `<project>`.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).
