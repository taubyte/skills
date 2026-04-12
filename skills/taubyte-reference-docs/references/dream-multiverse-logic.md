# Dream Multiverse Logic

## Multiverse

A Multiverse is the top-level container that manages multiple universes. It provides:
- Universe lifecycle management
- Persistent universe storage
- Context management
- Status monitoring

Location: `dream/multiverse.go`

## Creating a Multiverse

```go
import (
    "context"
    "github.com/taubyte/tau/dream"
)

ctx := context.Background()

// Create multiverse with default name
multiverse, err := dream.New(ctx)
if err != nil {
    return err
}
defer multiverse.Close()

// Create multiverse with custom name
multiverse, err := dream.New(ctx, dream.Name("my-multiverse"))
if err != nil {
    return err
}

// Create multiverse with persistent storage
multiverse, err := dream.New(ctx, 
    dream.LoadPersistent(),
    dream.Name("my-multiverse"),
)
```

## Multiverse Management

### Listing Universes

```go
// List all universes
names, err := multiverse.List()

// List persistent universes
persistent, err := multiverse.ListPersistent()

// List temporary universes
temporary, err := multiverse.ListTemporary()
```

### Getting Universe

```go
// Get a universe by name
universe, err := multiverse.Universe("my-universe")
```

### Getting Status

```go
// Get status of all universes
status := multiverse.Status()

// Status contains:
// - Root: Universe root directory
// - SwarmKey: P2P swarm key
// - NodeCount: Number of nodes
// - Simples: List of simple node names
// - Nodes: Map of node IDs to addresses
// - Services: List of services with copy counts
```

### Deleting Universe

```go
// Delete a universe from multiverse
err := multiverse.Delete("my-universe")
```

## Configuration

### Environment Variables

- `DREAM_CACHE_FOLDER`: Override cache folder location (default: `~/.cache/dream`)

### Constants

```go
// Default values
dream.DefaultHost = "127.0.0.1"
dream.DefaultMultiverseName = "default"
dream.DefaultUniverseName = "default"
dream.DreamApiPort = 1421

// Port configuration
dream.portStart = 100
dream.portBuffer = 21
```

### Cache Folder

```go
// Get cache folder (can be overridden)
cacheFolder, err := dream.GetCacheFolder("multiverse-name")
// Returns: ~/.cache/dream or ~/.cache/dream-{name}
```
