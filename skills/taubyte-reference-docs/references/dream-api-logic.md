# Dream API Logic

## HTTP API

Dream provides an HTTP API for managing universes programmatically.

Default Port: `1421` (configurable via `dream.DreamApiPort`)

Location: `dream/api/`

## Starting API Service

```go
import "github.com/taubyte/tau/dream/api"

// Create API service
apiService, err := api.New(multiverse, nil)
if err != nil {
    return err
}

// Start server
apiService.Server().Start()

// Wait for ready
ready, err := apiService.Ready(10 * time.Second)
```

## API Endpoints

Common endpoints include:
- `GET /status`: Get multiverse status
- `GET /universes`: List all universes
- `POST /universe`: Create universe
- `POST /universe/{name}/start`: Start universe
- `POST /universe/{name}/stop`: Stop universe
- `POST /universe/{name}/kill/{service}`: Kill service
- `GET /universe/{name}`: Get universe info
- `POST /universe/{name}/fixture/{name}`: Run fixture

See `dream/api/http_routes.go` for complete API documentation.

## MCP Integration

Dream provides MCP (Model Context Protocol) integration for AI-assisted development.

Location: `dream/mcp/`

MCP allows AI assistants to:
- Create and manage universes
- Start and stop services
- Run fixtures
- Query universe status
- Manage projects
