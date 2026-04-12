# Taubyte HTTP Function

## Function Code Pattern

**Function Responsibilities:**
- ✅ Receive input from Taubyte (via event.Event)
- ✅ Process business logic
- ✅ Return output (via event handlers)

### Package Declaration

```go
package lib  // MUST be "lib", not "main"
```

### Function Export

```go
//export functionName
func functionName(e event.Event) uint32 {
    // 1. GET INPUT (Taubyte provides this)
    // 2. DO LOGIC (Your business logic)
    // 3. PUT OUTPUT (Return to Taubyte)
    return 0  // 0 = success, 1+ = error
}
```

**Critical Requirements:**
- Package name must be `lib`
- `//export` comment must directly precede function
- Function name after `//export` must match function name in code
- Function receives `event.Event` and returns `uint32`
- Function handles ONLY business logic

## Basic HTTP Function

```go
package lib

import (
	"github.com/taubyte/go-sdk/event"
)

//export hello
func hello(e event.Event) uint32 {
	h, err := e.HTTP()
	if err != nil {
		return 1
	}

	h.Write([]byte("Hello from Taubyte!"))
	h.Return(200)
	return 0
}
```

## HTTP Function with JSON Parsing

```go
package lib

import (
	"encoding/json"
	"io"
	"github.com/taubyte/go-sdk/event"
)

//export createUser
func createUser(e event.Event) uint32 {
	h, err := e.HTTP()
	if err != nil {
		return 1
	}

	// Read request body
	body, err := io.ReadAll(h.Body())
	if err != nil {
		h.Write([]byte(`{"error": "Failed to read body"}`))
		h.Return(400)
		return 0
	}
	defer h.Body().Close()

	// Parse JSON
	var userData map[string]interface{}
	if err := json.Unmarshal(body, &userData); err != nil {
		h.Write([]byte(`{"error": "Invalid JSON"}`))
		h.Return(400)
		return 0
	}

	// Process user creation
	response := map[string]interface{}{
		"message": "User created successfully",
		"data":    userData,
	}
	jsonResponse, _ := json.Marshal(response)

	h.Headers().Set("Content-Type", "application/json")
	h.Write(jsonResponse)
	h.Return(201)
	return 0
}
```

## HTTP Function with Query Parameters

```go
package lib

import (
	"encoding/json"
	"github.com/taubyte/go-sdk/event"
)

//export search
func search(e event.Event) uint32 {
	h, err := e.HTTP()
	if err != nil {
		return 1
	}

	// Get query parameters
	query, _ := h.Query().Get("q")
	limit, _ := h.Query().Get("limit")

	response := map[string]interface{}{
		"query": query,
		"limit": limit,
	}

	jsonResponse, _ := json.Marshal(response)
	h.Headers().Set("Content-Type", "application/json")
	h.Write(jsonResponse)
	h.Return(200)
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

## Reading Request Body

```go
body, err := io.ReadAll(h.Body())
if err != nil {
    h.Write([]byte(`{"error": "Failed to read body"}`))
    h.Return(400)
    return 0
}
defer h.Body().Close()
```

## Reading Headers

```go
// Get specific header
userAgent, _ := h.UserAgent()
authHeader, _ := h.Headers().Get("Authorization")

// List all headers
allHeaders, _ := h.Headers().List()
```

## Reading Query Parameters

```go
// Get specific query parameter
query, _ := h.Query().Get("name")

// List all query parameters
allQueries, _ := h.Query().List()
```

## Writing Response

```go
// Set headers
h.Headers().Set("Content-Type", "application/json")

// Write response body
h.Write([]byte("response data"))

// Set status code
h.Return(200)
```

## CORS

CORS is built in; do not add custom CORS headers or OPTIONS handling.

## Key Takeaways

1. **Package Declaration**: Must use `package lib`, not `main`
2. **Function Export**: Use `//export functionName` comment directly above function
3. **Return Value**: Return `uint32` (0 = success, 1+ = error)
4. **HTTP Event Access**: Use `e.HTTP()` to get HTTP event handler. The type may be from `event` or `http/event`; both support the same methods.
5. **Resource Cleanup**: Always close `h.Body()` with `defer`
6. **Error Handling**: Return appropriate HTTP status codes; use JSON error bodies (e.g. `{"error": "message"}`) and avoid writing raw `err.Error()` to the response
