# Debugging Commands

## Debug Mode

Debug mode is configured when creating the universe. The multiverse must be created first.

What it does:
- Enables function call logging
- Creates `runtime.DebugFunctionCallsLogger`
- Writes logs to temp directory
- Captures all function execution details

Examples:
```bash
# Terminal 1: Create multiverse
dream new multiverse

# Terminal 2: Create universe (debug mode configured via universe creation)
dream new universe dev --enable auth,patrick,substrate
```

## Check Universe Status

```bash
dream status universe [name]
dream status u [name]
```

Output: Displays all nodes, protocols, and ports in the universe.

Examples:
```bash
dream status universe
dream status universe test
```

## Check Service Status

```bash
dream status service <service-name> [flags]
```

Flags:
- `--universe`, `-u` (string): Target universe name (default: "default")

Output: Shows service name, protocol, port, and accessible URL.

Examples:
```bash
dream status service patrick
dream status service substrate --universe test
```

## Get Universe ID

```bash
dream status id [name]
```

Output: Displays the universe ID.

Examples:
```bash
dream status id
dream status id test-universe
```

## Common Error Solutions

### "please provide a name"
```bash
dream new universe my-universe
dream new universe --name my-universe
```

### "parse arguments failed: write [arguments] after -flags"
```bash
dream new universe --enable auth my-universe
dream new universe my-universe --enable auth
```

### "can't set enable and disable flags"
```bash
dream new universe test --enable auth,patrick
dream new universe test --disable monkey,hoarder
```

### "attempted duplicate port bindings"
```bash
dream new universe test \
  --bind patrick@8080/http \
  --bind substrate@9000/p2p
```

### "universe is not in stopped state"
```bash
dream kill universe my-universe
dream new universe my-universe
```

### "failed getting service name 'X'"
```bash
dream status service patrick
dream status universe
dream inject service patrick
```

### "could not bind port of service `X`: disabled"
```bash
dream new universe test \
  --enable patrick \
  --bind patrick@8080/http
```
