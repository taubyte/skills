---
name: tb_cmd_tau_new_domain_generated_fqdn
description: Creates a domain resource with cloud-generated FQDN based on current context.
---

## Command card

**Preconditions:** Cloud context verified via `tau --defaults --yes json current`.

**Forbidden (unconditional):** Do **not** pass **`--generated-fqdn-prefix`**, **`--g-prefix`**, or any CLI synonym for a generated-FQDN prefix to **`tau new domain`** — **ever**, including if the user asks. See **`tb_sys_core_constraints`** and **`tb_sys_resource_flags_reference`** → *Domain generation*.

**Command:**

```bash
tau --defaults --yes new domain --name <domain> --generated-fqdn --type auto --description "<desc>"
```

**Postconditions:** Update context log.

**Critical note:** Generated FQDN shape depends on the currently selected cloud. Confirm remote context before creating domains intended for remote (`Cloud Type: remote`, expected cloud FQDN). Do not generate while Dream is selected when remote DNS behavior is required.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).
