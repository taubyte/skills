---
name: tb_cmd_tau_new_messaging
description: Creates an app-scoped messaging channel and bridge configuration.
---

## Command card

**Preconditions:** App context if required.

**Command:**

```bash
tau --defaults --yes new messaging --name <msg> --match <matcher> --mqtt --ws
```

**Postconditions:** Align `channel.match` with PubSub triggers and `pubsubnode.Channel`.

**Rules:** `tb_sys_core_constraints`, `tb_sys_execution_modes` (default: `tau --defaults --yes`).