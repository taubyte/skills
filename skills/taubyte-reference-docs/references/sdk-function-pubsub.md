# Taubyte PubSub Function

## Function Code Pattern

**Function Responsibilities:**

- ✅ Receive PubSub event from Taubyte (via event.Event)
- ✅ Process message data (business logic)
- ✅ Return success/error status

### Package Declaration

```go
package lib  // MUST be "lib", not "main"
```

### Function Export

```go
//export functionName
func functionName(e event.Event) uint32 {
    // 1. GET INPUT (Taubyte provides PubSub event)
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

## Channel Naming Rules

**CRITICAL**: Channel names have strict requirements:

- **MUST** use only lowercase letters
- **NO** hyphens, underscores, or symbols
- Example: `notifications` ✅
- Example: `notifications-channel` ❌
- Example: `Notifications` ❌
- Example: `notifications_channel` ❌

## CRITICAL — PubSub Event API (must match exactly)

- **`pubsubEvent.Data()` returns `([]byte, error)`.** Always assign to two variables (e.g. `data, err := pubsubEvent.Data()` or `_, _ = pubsubEvent.Data()`). Using a single variable causes compile error: "assignment mismatch: 1 variable but ... returns 2 values".
- **`pubsubEvent.Channel()` returns two values** — handle both.
- Reference: [github.com/taubyte/go-sdk](https://github.com/taubyte/go-sdk) — `pubsub/event/event.go`.

## PubSub Event Access

```go
pubsubEvent, err := e.PubSub()
if err != nil {
    return 1  // Not a PubSub event
}

// Data() returns ([]byte, error) — must use two variables
data, err := pubsubEvent.Data()
if err != nil {
    return 1
}
```

## PubSub Subscribe Function

```go
package lib

import (
	"github.com/taubyte/go-sdk/event"
)

//export subscribeHandler
func subscribeHandler(e event.Event) uint32 {
	// Get PubSub event
	pubsubEvent, err := e.PubSub()
	if err != nil {
		return 1
	}

	// Get channel and data
	channel, err := pubsubEvent.Channel()
	if err != nil {
		return 1
	}

	data, err := pubsubEvent.Data()
	if err != nil {
		return 1
	}

	// Process message based on channel
	switch channel {
	case "notifications":
		// Handle notification
	case "updates":
		// Handle update
	default:
		// Handle default
	}

	return 0
}
```

## Publishing Messages

### Channel Creation

```go
import "github.com/taubyte/go-sdk/pubsub"

// Channel name MUST be lowercase, no hyphens
channel, err := pubsub.Channel("notifications")
if err != nil {
    return 1
}
```

### Publishing Messages

```go
err = channel.Publish([]byte("message data"))
if err != nil {
    return 1
}
```

## PubSub Publish Function (from HTTP)

```go
package lib

import (
	"encoding/json"
	"io"
	"github.com/taubyte/go-sdk/event"
	"github.com/taubyte/go-sdk/pubsub"
)

//export publishMessage
func publishMessage(e event.Event) uint32 {
	h, err := e.HTTP()
	if err != nil {
		return 1
	}

	// Get channel name (MUST be lowercase, no hyphens)
	channelName, _ := h.Query().Get("channel")
	if channelName == "" {
		channelName = "notifications"
	}

	// Read message from body
	message, err := io.ReadAll(h.Body())
	if err != nil {
		h.Write([]byte(`{"error": "Failed to read message"}`))
		h.Return(400)
		return 0
	}
	defer h.Body().Close()

	// Get channel and publish
	channel, err := pubsub.Channel(channelName)
	if err != nil {
		h.Write([]byte(`{"error": "Failed to get channel"}`))
		h.Return(500)
		return 0
	}

	err = channel.Publish(message)
	if err != nil {
		h.Write([]byte(`{"error": "Failed to publish message"}`))
		h.Return(500)
		return 0
	}

	response := map[string]interface{}{
		"message": "Message published successfully",
		"channel": channelName,
	}
	jsonResponse, _ := json.Marshal(response)
	h.Headers().Set("Content-Type", "application/json")
	h.Write(jsonResponse)
	h.Return(200)
	return 0
}
```

## WebSocket URL Generation

For WebSocket support, use the `pubsub/node` package:

```go
import "github.com/taubyte/go-sdk/pubsub/node"

channel, err := pubsubnode.Channel("chat")
if err != nil {
    return 1
}

url, err := channel.WebSocket().Url()
if err != nil {
    return 1
}
```

## Subscribing to Channels

In a PubSub event handler:

```go
// In PubSub event handler
pubsubEvent, err := e.PubSub()
if err != nil {
    return 1
}

channel, err := pubsubEvent.Channel()
data, err := pubsubEvent.Data()
```

## Key Takeaways

1. **Channel Naming**: Channel names MUST be lowercase with no hyphens, underscores, or symbols
2. **Event Access**: Use `e.PubSub()` to get PubSub event handler
3. **Channel and Data**: Use `pubsubEvent.Channel()` and `pubsubEvent.Data()` to get message details
4. **Publishing**: Use `pubsub.Channel()` to get channel, then `channel.Publish()` to send messages
5. **WebSocket Support**: Use `pubsub/node` package for WebSocket URL generation
6. **Function Export**: Must use `//export` comment and `package lib`
