# Dream Simple Node Logic

## Simple Node

A Simple is a lightweight node that runs multiple service clients in a single process. Simple nodes are used for:
- Testing client interactions
- Running fixtures
- Development workflows
- Integration testing

Location: `dream/simple.go`

## Creating Simple Nodes

```go
// Create a simple node with default clients
config := &dream.SimpleConfig{
    Clients: universe.defaultClients(),
}
node, err := universe.CreateSimpleNode("my-client", config)

// Create a simple node with specific clients
config := &dream.SimpleConfig{
    Clients: map[string]*commonIface.ClientConfig{
        "auth":    {},
        "patrick": {},
        "tns":     {},
    },
}
node, err := universe.CreateSimpleNode("test-client", config)
```

## Accessing Simple Nodes

```go
// Get a simple node
simple, err := universe.Simple("client")

// Access clients from simple node
authClient, err := simple.Auth()
patrickClient, err := simple.Patrick()
tnsClient, err := simple.TNS()
seerClient, err := simple.Seer()
monkeyClient, err := simple.Monkey()
hoarderClient, err := simple.Hoarder()

// Get the peer node
peerNode := simple.PeerNode()
```

## Checking Client Availability

```go
// Check if clients are provided
err := simple.Provides("auth", "patrick", "tns")
if err != nil {
    // Some clients are not available
}
```

## Working with Simple Nodes

```go
// Create simple node
config := &dream.SimpleConfig{
    Clients: map[string]*commonIface.ClientConfig{
        "auth":    {},
        "patrick": {},
    },
}
node, err := universe.CreateSimpleNode("my-client", config)

// Get simple node
simple, err := universe.Simple("my-client")

// Use clients
authClient, err := simple.Auth()
patrickClient, err := simple.Patrick()
```
