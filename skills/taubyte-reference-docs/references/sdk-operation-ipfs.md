# Taubyte IPFS Operations (SDK)

## IPFS Client Creation

```go
import "github.com/taubyte/go-sdk/ipfs/client"

ipfsClient, err := ipfsclient.New()
if err != nil {
    return 1
}
```

## Creating and Storing Content

```go
// Create new content
content, err := ipfsClient.Create()
if err != nil {
    return 1
}
defer content.Close()

// Write data
_, err = content.Write([]byte("content data"))
if err != nil {
    return 1
}

// Push to IPFS and get CID
cid, err := content.Push()

// CID buffer size is 64 bytes
// Use fmt.Sprintf("%x", cid) to convert to hex string
```

## Retrieving Content by CID

```go
// Open content from IPFS (CID is []byte)
content, err := ipfsClient.Open(cidBytes)
if err != nil {
    return 1
}
defer content.Close()

// Read content
buffer := make([]byte, 1024)
n, err := content.Read(buffer)
```

## Complete IPFS Example

```go
package lib

import (
	"fmt"
	"io"
	"github.com/taubyte/go-sdk/event"
	"github.com/taubyte/go-sdk/ipfs/client"
)

//export storeIPFS
func storeIPFS(e event.Event) uint32 {
	h, err := e.HTTP()
	if err != nil {
		return 1
	}

	// Read data
	data, err := io.ReadAll(h.Body())
	if err != nil {
		h.Write([]byte(`{"error": "Failed to read data"}`))
		h.Return(400)
		return 0
	}
	defer h.Body().Close()

	// Create IPFS client
	ipfsClient, err := ipfsclient.New()
	if err != nil {
		h.Write([]byte(`{"error": "Failed to create IPFS client"}`))
		h.Return(500)
		return 0
	}

	// Create content
	content, err := ipfsClient.Create()
	if err != nil {
		h.Write([]byte(`{"error": "Failed to create IPFS content"}`))
		h.Return(500)
		return 0
	}
	defer content.Close()

	// Write data
	_, err = content.Write(data)
	if err != nil {
		h.Write([]byte(`{"error": "Failed to write to IPFS"}`))
		h.Return(500)
		return 0
	}

	// Push and get CID
	cid, err := content.Push()
	if err != nil {
		h.Write([]byte(`{"error": "Failed to push to IPFS"}`))
		h.Return(500)
		return 0
	}

	// Return CID
	response := fmt.Sprintf(`{"cid": "%x"}`, cid)
	h.Headers().Set("Content-Type", "application/json")
	h.Write([]byte(response))
	h.Return(200)
	return 0
}
```

## CID Format

- CID is `[]byte` (64 bytes)
- Convert to hex string: `fmt.Sprintf("%x", cid)`
- Convert to string representation: Use IPFS CID libraries if needed

## Resource Cleanup

Always close IPFS content:

```go
content, err := ipfsClient.Create()
defer content.Close()  // Always close

// ... operations ...

cid, err := content.Push()
```

## Key Takeaways

1. **Client Creation**: Use `ipfsclient.New()` to create IPFS client
2. **Content Creation**: Use `ipfsClient.Create()` to create new content
3. **Content Writing**: Use `content.Write()` to write data
4. **CID Retrieval**: Use `content.Push()` to save and get CID
5. **Content Reading**: Use `ipfsClient.Open(cidBytes)` to retrieve content
6. **CID Format**: CID is `[]byte` (64 bytes), convert with `fmt.Sprintf("%x", cid)`
7. **Resource Cleanup**: Always close content with `defer content.Close()`
