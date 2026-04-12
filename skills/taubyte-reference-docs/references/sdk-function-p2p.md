# Taubyte P2P Function

## Function Code Pattern

**Function Responsibilities:**
- ✅ Receive P2P event from Taubyte (via event.Event)
- ✅ Process command data (business logic)
- ✅ Return success/error status

### Package Declaration

```go
package lib  // MUST be "lib", not "main"
```

### Function Export

```go
//export functionName
func functionName(e event.Event) uint32 {
    // 1. GET INPUT (Taubyte provides P2P event)
    // 2. DO LOGIC (Your business logic)
    // 3. RETURN STATUS (Success/error)
    return 0  // 0 = success, 1+ = error
}
```

**Critical Requirements:**
- Package name must be `lib`
- `//export` comment must directly precede function
- Function name after `//export` must match function name in code
- Function receives `event.Event` and returns `uint32`
- Function handles ONLY business logic

## P2P Event Access

```go
p2pEvent, err := e.P2P()
if err != nil {
    return 1  // Not a P2P event
}
```

## Handling P2P Events

```go
p2pEvent, err := e.P2P()
if err != nil {
    return 1
}

protocol, err := p2pEvent.Protocol()
command, err := p2pEvent.Command()
data, err := p2pEvent.Data()
```

## Complete P2P Handler Example

```go
package lib

import (
	"github.com/taubyte/go-sdk/event"
)

//export p2pHandler
func p2pHandler(e event.Event) uint32 {
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

	// Process based on protocol and command
	switch protocol {
	case "myprotocol":
		switch command {
		case "ping":
			// Handle ping command
		case "pong":
			// Handle pong command
		default:
			// Handle unknown command
		}
	default:
		// Handle unknown protocol
	}

	return 0
}
```

## Key Takeaways

1. **Event Access**: Use `e.P2P()` to get P2P event handler
2. **Protocol and Command**: Use `p2pEvent.Protocol()` and `p2pEvent.Command()` to get details
3. **Data Access**: Use `p2pEvent.Data()` to get command data
4. **Function Export**: Must use `//export` comment and `package lib`
