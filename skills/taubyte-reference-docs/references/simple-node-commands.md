# Simple Node Commands

## Inject Simple Node

```bash
dream inject simple [name] [flags]
```

Flags:
- `--name`, `-n` (string): Simple node name (default: "client")
- `--universe`, `-u` (string): Target universe name (default: "default")
- `--enable` (string slice): Comma-separated list of clients to enable
- `--disable` (string slice): Comma-separated list of clients to disable
- `--empty` (bool): Create an empty simple node with no clients

Examples:
```bash
dream inject simple
dream inject simple my-client
dream inject simple test-client --enable auth,patrick,seer
dream inject simple minimal --disable monkey,hoarder
dream inject simple empty-node --empty
dream inject simple dev-client --universe test-universe
```

## Kill Simple Node

```bash
dream kill simple [name] [flags]
```

Flags:
- `--name`, `-n` (string): Simple node name (default: "client")
- `--universe`, `-u` (string): Target universe name (default: "default")

Examples:
```bash
dream kill simple
dream kill simple my-client
dream kill simple test-client --universe test-universe
```

## Common Patterns

### Default Simple Node
```bash
dream inject simple
dream inject simple client
```

### Minimal Simple Node
```bash
dream inject simple minimal --enable auth,patrick,seer
```

### Custom Client Configuration
```bash
dream inject simple custom \
  --enable auth,patrick,substrate,seer \
  --universe dev
```

### Empty Simple Node
```bash
dream inject simple empty-node --empty
```

### Multiple Simple Nodes
```bash
dream inject simple client1
dream inject simple client2
dream inject simple client3
```

### Simple Node with Exclusions
```bash
dream inject simple lightweight --disable monkey,hoarder
```
