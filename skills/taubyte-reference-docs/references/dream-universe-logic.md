# Dream Universe Logic

## Universe

A Universe is a complete local Tau deployment containing:
- All Tau services (Auth, Patrick, TNS, Seer, Monkey, Hoarder, Gateway, Substrate)
- Simple nodes (lightweight client nodes)
- P2P network mesh
- Isolated port space
- State management (Stopped, Starting, Running, Stopping)

Location: `dream/universe.go`

## Creating a Universe

```go
// Create a new universe
universe, err := multiverse.New(dream.UniverseConfig{
    Name:     "my-universe",
    KeepRoot: false, // Temporary universe (deleted on stop)
})

// Create a persistent universe
universe, err := multiverse.New(dream.UniverseConfig{
    Name:     "my-universe",
    KeepRoot: true, // Persists data between restarts
})
```

## Starting a Universe

### Start All Services (Default)

```go
// Start all services with default simple node named "client"
err := universe.StartAll()

// Start all services with custom simple nodes
err := universe.StartAll("client", "test-node")
```

### Start with Custom Configuration

```go
config := &dream.Config{
    Services: map[string]commonIface.ServiceConfig{
        "auth": {
            Others: map[string]int{
                "copies": 1, // Number of service instances
            },
        },
        "patrick": {
            Disabled: false,
        },
    },
    Simples: map[string]dream.SimpleConfig{
        "client": {
            Clients: map[string]*commonIface.ClientConfig{
                "auth":    {},
                "patrick": {},
                "tns":     {},
            },
        },
    },
}

err := universe.StartWithConfig(config)
```

## Stopping a Universe

```go
// Stop the universe (cleans up all services and nodes)
universe.Stop()

// Stop and close multiverse (stops all universes)
multiverse.Close()
```

## Universe States

Universe has four states managed through state machine:

```go
// Check universe state
state := universe.State()

// State check methods
if universe.Running() {
    // Universe is running
}

if universe.Stopped() {
    // Universe is stopped
}

if universe.Starting() {
    // Universe is starting
}

if universe.Stopping() {
    // Universe is stopping
}
```

States:
- `UniverseStateStopped`: Universe is not running
- `UniverseStateStarting`: Universe is starting services
- `UniverseStateRunning`: Universe is fully operational
- `UniverseStateStopping`: Universe is shutting down

## Network and Ports

### Port Management

```go
// Get port for a service and protocol
port, err := universe.PortFor("auth", "http")
port, err := universe.PortFor("patrick", "p2p")

// Get HTTP URL for a node
url, err := universe.GetURLHttp(node)
httpsUrl, err := universe.GetURLHttps(node)

// Get specific port for a node
httpPort, err := universe.GetPortHttp(node)
customPort, err := universe.GetPort(node, "p2p")
```

### Node Information

```go
// Get node information
info, err := universe.GetInfo(node)

// NodeInfo contains:
// - DbFactory: Database factory
// - Node: Peer node
// - Name: Node name
// - Ports: Map of protocol to port

// Lookup node by ID
nodeInfo, exists := universe.Lookup("node-id-string")

// Get all peers in universe
peers := universe.Peers()
allNodes := universe.All()
```

### Meshing

```go
// Mesh a new node with existing nodes
universe.Mesh(newNode)

// Nodes are automatically meshed when services start
```

## Universe Properties

```go
// Get universe properties
id := universe.Id()
root := universe.Root()
name := universe.Name()
persistent := universe.Persistent()
swarmKey := universe.SwarmKey()
context := universe.Context()
```

## Disk Usage

```go
// Get disk usage of universe (cached for 5 seconds)
size, err := universe.DiskUsage()

// Disk usage is automatically cached to avoid expensive calculations
```

## Killing Services and Nodes

### Killing Services

```go
// Kill a service (kills first instance)
err := universe.Kill("auth")

// Kill specific service instance by ID
err := universe.KillNodeByNameID("auth", "node-id")
```

### Killing Simple Nodes

```go
// Kill a simple node
err := universe.Kill("client")

// Kill specific simple node by ID
err := universe.KillNodeByNameID("client", "node-id")
```

## Common Patterns

### Basic Development Setup

```go
ctx := context.Background()

// Create multiverse
multiverse, err := dream.New(ctx, dream.LoadPersistent())
if err != nil {
    return err
}
defer multiverse.Close()

// Create universe
universe, err := multiverse.New(dream.UniverseConfig{
    Name:     "dev",
    KeepRoot: true,
})
if err != nil {
    return err
}

// Start all services
err = universe.StartAll()
if err != nil {
    return err
}

// Use services
auth := universe.Auth()
// ... work with services

// Stop when done
universe.Stop()
```

### Multiple Service Instances

```go
config := &dream.Config{
    Services: map[string]commonIface.ServiceConfig{
        "auth": {
            Others: map[string]int{
                "copies": 3, // Start 3 instances
            },
        },
    },
}

err := universe.StartWithConfig(config)

// Access all instances
allAuth := universe.AllAuth()
for _, auth := range allAuth {
    // Use each instance
}
```

### Custom Port Configuration

```go
config := &dream.Config{
    Services: map[string]commonIface.ServiceConfig{
        "auth": {
            Others: map[string]int{
                "http":   9000,
                "p2p":    9001,
                "ipfs":   9002,
                "dns":    9003,
                "copies": 1,
            },
        },
    },
}

err := universe.StartWithConfig(config)
```
