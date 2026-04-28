---
name: tb_cmd_tau_select_project
description: Selects the active project and verifies the selected context snapshot.
---

## Command card

**Preconditions:** Correct cloud/profile; intended project exists.

**Command:**

```bash
tau --defaults --yes select project --name <project>
tau --defaults --yes json current
```

**Postconditions:** Verify **Project** field matches intent; update `tb_sys_context_log`.

**Failure note:** If `project <name> not found`, re-check selected cloud/profile first (`tb_cmd_tau_select_cloud_fqdn` or `tb_cmd_tau_select_cloud_universe`), then retry.

---

## Mandatory after `tau new project`

Whenever **`tau new project --name <project>`** succeeds, run **`tau --defaults --yes select project --name <project>`** as the **very next** `tau` command (same turn), then **`tau --defaults --yes json current`**. Do **not** rely on `new project` having “already selected” the project — the profile is global and stale selections are the main cause of work attaching to the wrong GitHub project.

---

## End of “new project + resources” workflows (multi-app)

If you used **`tau select application`** for frontend/backend phases, before you **finish the task** (last `tau` / push / inject steps), run:

```bash
tau --defaults --yes clear application
tau --defaults --yes select project --name <project>
tau --defaults --yes json current
```

Confirm **Project** is `<project>`. That leaves the user’s CLI in a predictable state: correct project, no accidental application scope.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).
