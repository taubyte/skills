# Multiverse Commands

## Create Multiverse

```bash
dream new multiverse
```

Behavior:
- Creates a new Dream multiverse
- Must be created before creating universes
- Required first step for Dream environment setup
- **CRITICAL**: Must be run in a separate terminal or with daemon flag if available
- The multiverse runs continuously and must stay active while working with Dream

Examples:
```bash
dream new multiverse
```

**Note**: After creating the multiverse, it runs continuously. You can then create universes in another terminal or the same terminal after the multiverse is running.

## Common Patterns

### Basic Multiverse Setup
```bash
# Terminal 1: Create and run multiverse (keeps running)
dream new multiverse

# Terminal 2: Create universe (after multiverse is running)
dream new universe
```

### Development Setup
```bash
# Terminal 1: Create multiverse
dream new multiverse

# Terminal 2: Create development universe
dream new universe dev --enable auth,patrick,substrate
```

### Production-Like Setup
```bash
# Terminal 1: Create multiverse
dream new multiverse

# Terminal 2: Create production-like universe
dream new universe prod --enable auth,patrick,substrate,gateway
```

### Multiple Universes
```bash
# Terminal 1: Create multiverse
dream new multiverse

# Terminal 2: Create multiple universes
dream new universe test1
dream new universe test2
dream new universe test3
```

### Persistent Multiverse
```bash
# Terminal 1: Create multiverse (persists state)
dream new multiverse

# Terminal 2: Create universe
dream new universe persistent
```
