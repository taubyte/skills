---
name: spore-drive-sdk
description: Deploy Taubyte cloud using the @taubyte/spore-drive SDK. Use when the user wants to deploy a cloud, self-host Taubyte, or deploy on a domain/subdomain across servers. Creates deployment code from scratch; infers inputs from natural messages; prompts only for missing required values.
---

# Spore Drive SDK — Deploy Taubyte Cloud

Use this skill when the user wants to deploy a Taubyte cloud (PaaS/IDP) across servers. The [@taubyte/spore-drive](https://www.npmjs.com/package/@taubyte/spore-drive) SDK lets you define and deploy cloud infrastructure as code.

## Intent Recognition

Apply when the user says things like:

- "deploy a cloud"
- "deploy taubyte cloud"
- "self-host taubyte"
- "deploy on domain X and subdomain Y on these servers [list]"
- "deploy a cloud on pom.ac and g.pom.ac on node1,203.0.113.1 and node2,203.0.113.2"

## Input Handling

- **Extract from message**: Domain (ROOT_DOMAIN), subdomain (GENERATED_DOMAIN), servers (hostname + public_ip, optionally location). Use whatever the user provides.
- **Prompt when missing**: If required inputs are missing (SSH key path, Tau choice, optional DNS automation), **ask**. **SSH user:** do not ask upfront — use `ubuntu` first, then `root` if deployment fails; only ask the user for their username if both fail. You are obliged to prompt for other missing values — never fail silently or guess.
- **Natural phrasing**: e.g. "What's the path to your SSH key?" not a rigid form.

## Required Inputs

| Input | Description |
|-------|-------------|
| ROOT_DOMAIN | Root domain (e.g. `pom.ac`) |
| GENERATED_DOMAIN | Generated subdomain (e.g. `g.pom.ac`) |
| Servers | `hostname,public_ip` per server |
| SSH_KEY | Path to SSH private key |
| SSH_USER | Default `ubuntu`, then `root` if that fails; ask user only if both fail |
| Tau | `latest` or `custom` (path if custom) |
| DNS automation | Optional — Namecheap supported via API; other providers use manual DNS |

See `references/spore-drive-cloud-deployment.md` in this skill for full input reference and DNS instructions.

## SDK API Reference

### Imports

```ts
import { Config, Drive, TauLatest, TauPath, CourseConfig, Course } from "@taubyte/spore-drive";
```

### Config

```ts
const config = new Config("/path/to/config");  // or new Config() for in-memory
await config.init();

// Cloud domain
await config.cloud.domain.root.set(process.env.ROOT_DOMAIN || "pom.ac");
await config.cloud.domain.generated.set(process.env.GENERATED_DOMAIN || "g.pom.ac");
await config.cloud.domain.validation.generate();  // if keys not yet set
await config.cloud.p2p.swarm.generate();

// Auth (SSH key)
const mainAuth = config.auth.signer["main"];
await mainAuth.username.set(process.env.SSH_USER || "ubuntu");
await mainAuth.key.path.set("keys/ssh-key.pem");
await mainAuth.key.data.set(await fs.promises.readFile(process.env.SSH_KEY!));

// Shapes
const all = config.shapes.get("all");
await all.services.set(["auth", "tns", "hoarder", "seer", "substrate", "patrick", "monkey"]);
await all.ports.port["main"].set(4242);
await all.ports.port["lite"].set(4262);

// Hosts (from CSV or list)
for (const { hostname, publicIp } of servers) {
  const host = config.hosts.get(hostname);
  await host.addresses.add([`${publicIp}/32`]);
  await host.ssh.address.set(`${publicIp}:22`);
  await host.ssh.auth.add(["main"]);
  await host.location.set("40.730610, -73.935242");  // or user-provided
  if (!(await host.shapes.list()).includes("all"))
    await host.shapes.get("all").generate();
}

// P2P bootstrap
await config.cloud.p2p.bootstrap.shape["all"].nodes.add(bootstrapHostnames);

await config.commit();
```

### Drive and Course

```ts
const tauBinary = process.env.TAU_PATH ? TauPath(process.env.TAU_PATH) : TauLatest;
const drive = new Drive(config, tauBinary);
await drive.init();

const course = await drive.plot(new CourseConfig(["all"]));
await course.displace();

// Headless progress (no TTY) — log as text
for await (const d of await course.progress()) {
  const host = (d.path.match(/\/([^/]+):\d+/) || [null, "?"])[1];
  const task = d.path.split("/").pop() || "running";
  console.log(`[${host}] [${task}] [${Math.round(d.progress)}%]`);
  if (d.error) throw new Error(d.error);
}
```

### Headless / Non-TTY

For Cursor's embedded terminal: **do not** use progress bars (e.g. `@opentf/cli-pbar`). Log progress as lines: `[hostname] [task] [pct%]`.

## Project Structure

Create a deploy folder (e.g. `spore-drive-deploy/`):

```
spore-drive-deploy/
├── package.json     # @taubyte/spore-drive, dotenv, tsx
├── src/deploy.ts    # or index.ts
├── hosts.csv        # hostname,public_ip
├── .env             # ROOT_DOMAIN, GENERATED_DOMAIN, SSH_KEY, SSH_USER, TAU_PATH (opt)
└── .env.example
```

**package.json** scripts:

```json
"scripts": { "displace": "tsx src/deploy.ts" }
```

**Run**: `npm install` then `npm run displace` — do this in the background; do not ask the user to run commands.

## Flow

1. Infer domain, subdomain, servers from user message.
2. Prompt for any missing required inputs (except SSH user — use ubuntu first, then root if deploy fails).
3. Create deploy project, write code using the SDK, generate `hosts.csv` and `.env`.
4. Run `npm install` and `npm run displace` from the deploy folder. Use `SSH_USER=ubuntu` by default. If displacement fails with SSH/auth errors, retry with `SSH_USER=root`. If that fails, ask the user for their SSH username.
5. If no automated DNS (or non-Namecheap): provide manual DNS instructions for any provider (seer A records, tau NS, wildcard CNAME).
