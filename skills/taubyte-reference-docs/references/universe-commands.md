# Universe Commands

## Create Universe

```bash
dream new universe [name] [flags]
```

**Important**: You must create a multiverse first with `dream new multiverse` before creating universes.

Flags:
- `--name`, `-n` (string): Universe name (default: "default")
- `--empty` (bool): Create an empty universe with no services or simples (overrides other flags)
- `--enable` (string slice): Comma-separated list of services to enable (conflicts with --disable)
- `--disable` (string slice): Comma-separated list of services to disable (conflicts with --enable)
- `--bind` (string slice): Service binding configuration in format `service@port/protocol`
- `--fixtures` (string slice): Comma-separated list of fixtures to run after universe creation
- `--simples` (string slice): Comma-separated list of simple node names (default: "client")

Examples:
```bash
dream new multiverse
dream new universe
dream new universe my-universe
dream new universe test --empty
dream new universe minimal --enable auth,patrick,substrate
dream new universe no-monkey --disable monkey
dream new universe custom --bind patrick@8080/http,substrate@9000/p2p
dream new universe test --fixtures set-branch,push-all
dream new universe multi --simples client1,client2,client3
```

## Check Universe Status

```bash
dream status universe [name]
dream status u [name]
```

Examples:
```bash
dream status universe
dream status universe my-universe
```

## Kill Universe

```bash
dream kill universe [name]
```

Examples:
```bash
dream kill universe
dream kill universe my-universe
```

## Common Patterns

### Minimal Universe
```bash
dream new multiverse
dream new universe minimal --enable auth,patrick,substrate
```

### Full-Stack Universe
```bash
dream new multiverse
dream new universe full-stack
```

### Custom Service Configuration
```bash
dream new multiverse
dream new universe custom \
  --bind patrick@8080/http \
  --bind substrate@9000/p2p \
  --bind gateway@443/https
```

### Development Universe with Fixtures
```bash
dream new multiverse
dream new universe dev \
  --fixtures set-branch,push-all \
  --simples test-client
```
