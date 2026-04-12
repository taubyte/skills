# Taubyte P2P Operations (SDK)

## Creating P2P Service

```go
import "github.com/taubyte/go-sdk/p2p/node"

service, err := node.New("myprotocol")
if err != nil {
    return 1
}
```

## Starting to Listen

```go
// Start listening
err = service.Listen()
if err != nil {
    return 1
}
```

## Sending P2P Commands

```go
command, err := service.Command("ping")
if err != nil {
    return 1
}

err = command.Send([]byte("command data"))
if err != nil {
    return 1
}
```

## Handling P2P Events

In a P2P function handler:

```go
p2pEvent, err := e.P2P()
if err != nil {
    return 1
}

protocol, err := p2pEvent.Protocol()
command, err := p2pEvent.Command()
data, err := p2pEvent.Data()
```

## Complete P2P Service Example

```go
package lib

import (
	"github.com/taubyte/go-sdk/p2p/node"
)

//export startP2PService
func startP2PService(e event.Event) uint32 {
	// Create P2P service
	service, err := node.New("myprotocol")
	if err != nil {
		return 1
	}

	// Start listening
	err = service.Listen()
	if err != nil {
		return 1
	}

	// Service is now listening for commands
	return 0
}
```

## Complete P2P Command Handler

```go
package lib

import (
	"github.com/taubyte/go-sdk/event"
)

//export p2pCommandHandler
func p2pCommandHandler(e event.Event) uint32 {
	// Get P2P event
	p2pEvent, err := e.P2P()
	if err != nil {
		return 1
	}

	// Get protocol, command, and data
	protocol, err := p2pEvent.Protocol()
	if err != nil {
		return 1
	}

	command, err := p2pEvent.Command()
	if err != nil {
		return 1
	}

	data, err := p2pEvent.Data()
	if err != nil {
		return 1
	}

	// Process command based on protocol
	switch protocol {
	case "myprotocol":
		switch command {
		case "ping":
			// Handle ping
		case "pong":
			// Handle pong
		}
	}

	return 0
}
```

## Key Takeaways

1. **Service Creation**: Use `node.New("protocol")` to create P2P service
2. **Listening**: Use `service.Listen()` to start listening for commands
3. **Sending Commands**: Use `service.Command("name")` and `command.Send(data)` to send commands
4. **Event Handling**: Use `e.P2P()` to get P2P event in function handlers
5. **Protocol and Command**: Use `p2pEvent.Protocol()` and `p2pEvent.Command()` to get details
6. **Data Access**: Use `p2pEvent.Data()` to get command data
