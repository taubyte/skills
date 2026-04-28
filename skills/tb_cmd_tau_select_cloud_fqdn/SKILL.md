---
name: tb_cmd_tau_select_cloud_fqdn
description: Selects a remote cloud FQDN and verifies context is remote before mutations.
---

## Command card

**Preconditions:** `tb_sys_cli_prereqs` passed; user supplied FQDN.

**Command:**

```bash
tau --defaults --yes select cloud --fqdn <cloud_fqdn>
tau --defaults --yes json current
```

**Postconditions:** Log cloud target; run `tb_cmd_tau_json_current`.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).