# Taubyte HTTP Operations (SDK)

## CRITICAL — go-sdk HTTP API (must match exactly)

- **`h.Headers().Get(key)` returns `(string, error)`.** Always assign to two variables (e.g. `val, _ := h.Headers().Get("X-Name")` or `val, err := ...`). Using a single variable causes compile error: "assignment mismatch: 1 variable but ... returns 2 values".
- **`h.Query().Get(key)` returns `(string, error)`.** Same rule: use two variables. Use `h.Query().Get("name")` for query params.
- **There is no `h.URL()`.** The HTTP event type has no `URL()` method. Do not use `h.URL().Query().Get(...)` — use `h.Query().Get(...)` only.
- Reference: [github.com/taubyte/go-sdk](https://github.com/taubyte/go-sdk) — `http/event/header.go`, `http/event/query.go`.

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

### Get Specific Header

```go
userAgent, _ := h.UserAgent()
authHeader, _ := h.Headers().Get("Authorization")
```

### List All Headers

```go
allHeaders, _ := h.Headers().List()
```

## Reading Query Parameters

### Get Specific Query Parameter

```go
query, _ := h.Query().Get("name")
```

### List All Query Parameters

```go
allQueries, _ := h.Query().List()
```

## Writing Response

### Set Headers

```go
h.Headers().Set("Content-Type", "application/json")
```

### Write Response Body

```go
h.Write([]byte("response data"))
```

### Set Status Code

```go
h.Return(200)
```

## CORS

CORS is built in; do not add custom CORS headers or OPTIONS handling in your handlers.

## Complete HTTP Handler Example

```go
package lib

import (
	"encoding/json"
	"io"
	"github.com/taubyte/go-sdk/event"
)

//export httpHandler
func httpHandler(e event.Event) uint32 {
	h, err := e.HTTP()
	if err != nil {
		return 1
	}

	// Get method
	method, _ := h.Method()

	// Read body for POST/PUT
	if method == "POST" || method == "PUT" {
		body, err := io.ReadAll(h.Body())
		if err != nil {
			h.Write([]byte(`{"error": "Failed to read body"}`))
			h.Return(400)
			return 0
		}
		defer h.Body().Close()

		// Parse JSON
		var data map[string]interface{}
		json.Unmarshal(body, &data)
	}

	// Get query parameters
	query, _ := h.Query().Get("q")

	// Response
	response := map[string]interface{}{
		"method": method,
		"query":  query,
	}
	jsonResponse, _ := json.Marshal(response)
	h.Headers().Set("Content-Type", "application/json")
	h.Write(jsonResponse)
	h.Return(200)
	return 0
}
```

## Key Takeaways

1. **Request Body**: Use `io.ReadAll(h.Body())` and always close with `defer`
2. **Headers**: Use `h.Headers().Get()` and `h.Headers().Set()`
3. **Query Parameters**: Use `h.Query().Get()` to get query values
4. **Response**: Use `h.Write()` for body, `h.Return()` for status code
5. **CORS**: Built in — do not add custom CORS headers or OPTIONS handling
6. **Content-Type**: Always set `Content-Type` header for JSON responses
