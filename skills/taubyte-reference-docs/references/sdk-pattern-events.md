# Taubyte Event Handling

## Event Type Checking

```go
import "github.com/taubyte/go-sdk/event"

func handler(e event.Event) uint32 {
    eventType := e.Type()
    
    switch eventType {
    case "http":
        // Handle HTTP event
    case "pubsub":
        // Handle PubSub event
    case "p2p":
        // Handle P2P event
    }
    return 0
}
```

## HTTP Event Access

```go
h, err := e.HTTP()
if err != nil {
    return 1  // Not an HTTP event
}
```

## PubSub Event Access

```go
pubsubEvent, err := e.PubSub()
if err != nil {
    return 1  // Not a PubSub event
}
```

## P2P Event Access

```go
p2pEvent, err := e.P2P()
if err != nil {
    return 1  // Not a P2P event
}
```

## Complete Event Handler Example

```go
package lib

import (
	"github.com/taubyte/go-sdk/event"
)

//export multiEventHandler
func multiEventHandler(e event.Event) uint32 {
	eventType := e.Type()
	
	switch eventType {
	case "http":
		// Handle HTTP event
		h, err := e.HTTP()
		if err != nil {
			return 1
		}
		h.Write([]byte("HTTP event"))
		h.Return(200)
		
	case "pubsub":
		// Handle PubSub event
		pubsubEvent, err := e.PubSub()
		if err != nil {
			return 1
		}
		channel, _ := pubsubEvent.Channel()
		data, _ := pubsubEvent.Data()
		// Process message
		
	case "p2p":
		// Handle P2P event
		p2pEvent, err := e.P2P()
		if err != nil {
			return 1
		}
		protocol, _ := p2pEvent.Protocol()
		command, _ := p2pEvent.Command()
		data, _ := p2pEvent.Data()
		// Process command
	}
	
	return 0
}
```

## Event Flow

```
Event Occurs → Taubyte Routes Event → Function Processes → Response Delivered
```

## Key Takeaways

1. **Event Type**: Use `e.Type()` to check event type
2. **HTTP Events**: Use `e.HTTP()` to get HTTP event handler
3. **PubSub Events**: Use `e.PubSub()` to get PubSub event handler
4. **P2P Events**: Use `e.P2P()` to get P2P event handler
5. **Error Handling**: Always check for errors when accessing event types
6. **Event-Driven**: Functions respond to events, not direct calls
