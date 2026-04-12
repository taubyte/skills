# Taubyte Error Handling & Best Practices

## Standard Error Response

Return **JSON** error payloads (e.g. `{"error": "message"}`). Do **not** write raw `err.Error()` to the response—it can leak internal details and is not valid JSON.

```go
func handleError(h event.HttpEvent, message string, code int) uint32 {
	h.Write([]byte(fmt.Sprintf(`{"error": "%s"}`, message)))
	h.Return(code)
	return 0
}
```

**HTTP event type:** The handler may use `event.HttpEvent` (from `github.com/taubyte/go-sdk/event`) or `http.Event` (from `github.com/taubyte/go-sdk/http/event`); both support the same methods (Write, Return, Headers, Body, Query, etc.).

## Error Handling in Functions

```go
func myFunction(e event.Event) uint32 {
	h, err := e.HTTP()
	if err != nil {
		return 1
	}

	// Validate input
	if invalid {
		return handleError(h, "Invalid input", 400)
	}

	// Process
	if processError != nil {
		return handleError(h, "Processing failed", 500)
	}

	// Success
	h.Write([]byte(`{"status": "success"}`))
	h.Return(200)
	return 0
}
```

## Resource Cleanup Patterns

### Always Close Resources

```go
// Storage content
content, err := stor.New().File("file.txt")
if err != nil {
	return 1
}
defer content.Close()

// HTTP body
defer h.Body().Close()

// IPFS content
ipfsContent, err := ipfsClient.Create()
if err != nil {
	return 1
}
defer ipfsContent.Close()
```

## JSON Handling Patterns

### Request Parsing

```go
body, err := io.ReadAll(h.Body())
if err != nil {
	return handleError(h, "Failed to read body", 400)
}
defer h.Body().Close()

var data map[string]interface{}
if err := json.Unmarshal(body, &data); err != nil {
	return handleError(h, "Invalid JSON", 400)
}
```

### Response Generation

```go
response := map[string]interface{}{
	"status":  "success",
	"data":    result,
	"message": "Operation completed",
}
jsonResponse, _ := json.Marshal(response)
h.Headers().Set("Content-Type", "application/json")
h.Write(jsonResponse)
h.Return(200)
```

## File Upload/Download Patterns

### Upload Pattern

```go
// Read file data
fileData, err := io.ReadAll(h.Body())
defer h.Body().Close()

// Get filename
filename, _ := h.Query().Get("filename")

// Create and write file
content, err := stor.New().File(filename)
defer content.Close()
content.Write(fileData)
cid, err := content.Push()
```

### Download Pattern

```go
// Get file
file, err := stor.File(filename)
defer file.Close()

// Read file
buffer := make([]byte, 1024)
var fileData []byte
for {
	n, err := file.Read(buffer)
	if n > 0 {
		fileData = append(fileData, buffer[:n]...)
	}
	if err == io.EOF {
		break
	}
}

// Return file
h.Headers().Set("Content-Type", "application/octet-stream")
h.Write(fileData)
```

## Webhook Processing Patterns

```go
// Read payload
payload, err := io.ReadAll(h.Body())
defer h.Body().Close()

// Validate signature (if needed)
signature, _ := h.Headers().Get("X-Signature")
if !validateSignature(payload, signature) {
	return handleError(h, "Invalid signature", 401)
}

// Store payload
stor.File("webhook.json").Add(payload, true)

// Publish notification
channel.Publish(payload)

// Return acknowledgment
h.Write([]byte(`{"status": "received"}`))
h.Return(200)
```

## Data Pipeline Patterns

```go
// Step 1: Store raw data
stor.File("raw.json").Add(rawData, true)

// Step 2: Process data
processedData := process(rawData)

// Step 3: Store processed data
db.Put("processed", processedData)

// Step 4: Publish event
channel.Publish(processedData)

// Step 5: Archive to IPFS
ipfsContent.Write(processedData)
ipfsContent.Push()
```

## Multi-Service Integration Patterns

```go
// Use storage - pass matcher from config (match: ...), NOT resource name
stor, _ := storage.Get("uploads")

// Use database - pass matcher from config (match: ...), NOT resource name
db, _ := database.New("notes")

// Use PubSub
channel, _ := pubsub.Channel("notifications")

// Use IPFS
ipfsClient, _ := ipfsclient.New()

// Use HTTP client
httpClient, _ := httpclient.New()
```

## Key Takeaways

1. **Error Handling**: Use consistent error response patterns; return JSON (e.g. `{"error": "message"}`) and avoid writing raw `err.Error()` to the response
2. **Resource Cleanup**: Always use `defer` to close resources
3. **JSON Parsing**: Validate JSON and handle errors
4. **File Operations**: Use proper read/write patterns with cleanup
5. **Webhook Processing**: Validate signatures and acknowledge receipt
6. **Data Pipelines**: Chain operations with proper error handling
7. **Multi-Service**: Integrate multiple services with error handling
