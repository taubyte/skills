# Function Logic

## Function Types

### HTTP Functions
HTTP functions are triggered by HTTP requests.

Configuration:
- Methods: HTTP methods (GET, POST, PUT, DELETE, PATCH, etc.)
- Domains: Domain names that can trigger the function
- Paths: URL paths that trigger the function
- Timeout: Request timeout duration
- Memory: Memory allocation for execution

### P2P Functions
P2P functions are triggered by peer-to-peer protocol commands.

Configuration:
- Protocol: P2P protocol/service name
- Command: Command name that triggers the function
- Local: Enable local execution
- Timeout: Execution timeout
- Memory: Memory allocation

### PubSub Functions
PubSub functions are triggered by messages on pub/sub channels.

Configuration:
- Channel: Channel name to subscribe to
- Local: Enable local execution
- Timeout: Execution timeout
- Memory: Memory allocation

## Function Configuration

### Timeout
- Maximum execution time for the function
- Format: duration string (e.g., "30s", "5m", "1h")
- Default varies by function type

### Memory
- Memory allocation for function execution
- Can be specified with unit: `--memory 512 --memory-unit MB`
- Or as combined: `--memory 512MB`
- Units: MB, GB

### Source Code
- Path to function source code
- Entry point specifies the function to execute
- Supports various languages (configured in project)

### Entry Point
- Function name or method to execute
- Default varies by language
- Required for most function types

## Function Execution

### HTTP Function Execution
- Triggered by HTTP requests matching configured domains and paths
- Receives HTTP request context
- Returns HTTP response

### P2P Function Execution
- Triggered by P2P protocol commands
- Receives command data
- Can execute locally or remotely

### PubSub Function Execution
- Triggered by messages on subscribed channel
- Receives message payload
- Can execute locally or remotely
