---
name: tb_cmd_tau_query_domain
description: Queries a domain resource by name to retrieve effective FQDN and registration details.
---

## Command card

**Preconditions:** Domain resource exists.

**Command:**

```bash
tau --defaults --yes query domain --name <domain-name>
```

**Postconditions:** Use FQDN for hosts + curl (see `tb_sys_local_host_launch`).

**Scope note:** If the domain is project-scoped and query fails while an application is selected, clear app scope and retry:

```bash
tau --defaults --yes clear application
tau --defaults --yes query domain --name <domain-name>
```

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).