# Taubyte Database Operations (SDK)

## Matcher vs Resource Name

**Use the matcher (match pattern), NOT the resource name.** The SDK resolves databases by the `match` value from the database config (`config/databases/[name].yaml`). The config file name (e.g. `noting14_db`) is the resource name; the `match:` field (e.g. `notes`) is the matcher. In code, always pass the matcher to `database.New()`.

**Use path-style keys, not colon-style.** Use slashes for key namespaces (e.g. `user/123`, `order/789`, `user/123/profile`). Do not use colons (e.g. `user:123`).

## Creating Database

```go
import "github.com/taubyte/go-sdk/database"

// Use the match value from config (e.g. match: notes), NOT the resource name (e.g. noting14_db)
db, err := database.New("notes")
if err != nil {
    return 1
}
```

## Database Operations

### Put Data

```go
err = db.Put("key", []byte("value"))
```

### Get Data

```go
value, err := db.Get("key")
if err != nil {
    // Key not found
}
```

### Delete Data

```go
err = db.Delete("key")
```

### List Keys

```go
// List all keys (e.g. for "get all" or leaderboard endpoints)
keys, err := db.List("")
if err != nil {
    // Handle error
}

// List keys under a prefix (use path-style, e.g. "user/")
keys, err := db.List("user/")
if err != nil {
    // Handle error
}
// keys is a []string of key names
```

## Complete Database Example

```go
package lib

import (
	"encoding/json"
	"io"
	"github.com/taubyte/go-sdk/event"
	"github.com/taubyte/go-sdk/database"
)

//export storeData
func storeData(e event.Event) uint32 {
	h, err := e.HTTP()
	if err != nil {
		return 1
	}

	// Get database - use matcher from config (match: ...), not resource name
	db, err := database.New("notes")
	if err != nil {
		h.Write([]byte(`{"error": "Failed to get database"}`))
		h.Return(500)
		return 0
	}

	// Get key from query
	key, _ := h.Query().Get("key")
	if key == "" {
		h.Write([]byte(`{"error": "Key required"}`))
		h.Return(400)
		return 0
	}

	// Read value from body
	value, err := io.ReadAll(h.Body())
	if err != nil {
		h.Write([]byte(`{"error": "Failed to read value"}`))
		h.Return(400)
		return 0
	}
	defer h.Body().Close()

	// Store data
	err = db.Put(key, value)
	if err != nil {
		h.Write([]byte(`{"error": "Failed to store data"}`))
		h.Return(500)
		return 0
	}

	response := map[string]interface{}{
		"message": "Data stored successfully",
		"key":     key,
		"size":    len(value),
	}
	jsonResponse, _ := json.Marshal(response)
	h.Headers().Set("Content-Type", "application/json")
	h.Write(jsonResponse)
	h.Return(200)
	return 0
}
```

## JSON Data Storage

### Storing JSON

```go
data := map[string]interface{}{
	"name":  "John",
	"email": "john@example.com",
}
jsonData, _ := json.Marshal(data)
err = db.Put("user/123", jsonData)
```

### Retrieving JSON

```go
jsonData, err := db.Get("user/123")
if err != nil {
	// Key not found
	return
}

var user map[string]interface{}
json.Unmarshal(jsonData, &user)
```

## Key-Value Patterns

### Simple Key-Value

```go
db.Put("key", []byte("value"))
value, _ := db.Get("key")
```

### Namespaced Keys

```go
db.Put("user/123", userData)
db.Put("user/456", userData)
db.Put("order/789", orderData)
```

### Composite Keys

```go
key := fmt.Sprintf("user/%s/profile", userID)
db.Put(key, profileData)
```

## Error Handling

### Key Not Found

```go
value, err := db.Get("nonexistent")
if err != nil {
	// Handle key not found
}
```

### Database Errors

```go
err = db.Put("key", []byte("value"))
if err != nil {
	// Handle database error
	return 1
}
```

## Key Takeaways

1. **Database Creation**: Use `database.New(match)` with the **matcher** from config (`match:` field), NOT the resource name
2. **Put Operation**: Store data with `db.Put(key, []byte(value))`
3. **Get Operation**: Retrieve data with `db.Get(key)`
4. **Delete Operation**: Remove data with `db.Delete(key)`
5. **List Operation**: List key names with `db.List("")` (all keys) or `db.List("prefix/")` (keys under prefix); useful for "get all" or leaderboard-style endpoints
6. **JSON Storage**: Marshal/unmarshal JSON for structured data
7. **Key Patterns**: Use path-style keys (e.g., `user/123`, `order/789`), not colon-style (`user:123`)
8. **Error Handling**: Always check for errors, especially on `Get()` and `List()` operations
