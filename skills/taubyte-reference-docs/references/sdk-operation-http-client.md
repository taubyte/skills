# Taubyte HTTP Client Operations

## Creating HTTP Client

```go
import "github.com/taubyte/go-sdk/http/client"

httpClient, err := httpclient.New()
if err != nil {
    return 1
}
```

## Making Requests

### GET Request

```go
response, err := httpClient.Get("https://api.example.com/data")
if err != nil {
    return 1
}
defer response.Body().Close()

body, err := io.ReadAll(response.Body())
statusCode := response.StatusCode()
```

### POST Request

```go
response, err := httpClient.Post("https://api.example.com/data", []byte("post data"))
if err != nil {
    return 1
}
defer response.Body().Close()
```

### PUT Request

```go
response, err := httpClient.Put("https://api.example.com/data", []byte("put data"))
```

### DELETE Request

```go
response, err := httpClient.Delete("https://api.example.com/data")
```

## Complete HTTP Client Example

```go
package lib

import (
	"io"
	"github.com/taubyte/go-sdk/event"
	"github.com/taubyte/go-sdk/http/client"
)

//export callExternalAPI
func callExternalAPI(e event.Event) uint32 {
	h, err := e.HTTP()
	if err != nil {
		return 1
	}

	// Create HTTP client
	httpClient, err := httpclient.New()
	if err != nil {
		h.Write([]byte(`{"error": "Failed to create HTTP client"}`))
		h.Return(500)
		return 0
	}

	// Make GET request
	response, err := httpClient.Get("https://api.example.com/data")
	if err != nil {
		h.Write([]byte(`{"error": "Failed to make request"}`))
		h.Return(500)
		return 0
	}
	defer response.Body().Close()

	// Read response
	body, err := io.ReadAll(response.Body())
	if err != nil {
		h.Write([]byte(`{"error": "Failed to read response"}`))
		h.Return(500)
		return 0
	}

	// Get status code
	statusCode := response.StatusCode()

	// Return response
	h.Headers().Set("Content-Type", "application/json")
	h.Write(body)
	h.Return(statusCode)
	return 0
}
```

## Response Handling

### Read Response Body

```go
body, err := io.ReadAll(response.Body())
defer response.Body().Close()
```

### Get Status Code

```go
statusCode := response.StatusCode()
```

## Key Takeaways

1. **Client Creation**: Use `httpclient.New()` to create HTTP client
2. **GET Requests**: Use `httpClient.Get(url)`
3. **POST Requests**: Use `httpClient.Post(url, data)`
4. **PUT Requests**: Use `httpClient.Put(url, data)`
5. **DELETE Requests**: Use `httpClient.Delete(url)`
6. **Response Handling**: Always close response body with `defer response.Body().Close()`
7. **Status Codes**: Use `response.StatusCode()` to get HTTP status code
