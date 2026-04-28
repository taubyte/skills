---
name: tb_cmd_tau_new_domain_fqdn
description: Creates a domain resource using an explicit custom FQDN and certificate mode.
---

## Command card

**Preconditions:** Project context; cloud context verified with `tau --defaults --yes json current`.

**Command:**

```bash
tau --defaults --yes new domain --name <domain> --fqdn <fqdn> --type auto --description "<desc>"
```

**Postconditions:** Update context log. Git Bash on Windows: prefix with `MSYS_NO_PATHCONV=1` when flags look like paths.

**Warning:** Do not hand-craft `*.default.<cloud-fqdn>` hostnames unless DNS delegation/TXT proof is known to exist. Prefer `--generated-fqdn` for cloud-managed generated hosts.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).