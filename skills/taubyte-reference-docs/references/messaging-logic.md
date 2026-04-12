# Messaging Logic

## Messaging Protocols

### MQTT
- Message Queuing Telemetry Transport protocol
- Lightweight publish-subscribe messaging
- Enable with `--mqtt` flag
- Suitable for IoT and real-time messaging

### WebSocket
- WebSocket protocol support
- Full-duplex communication
- Enable with `--websocket` flag
- Suitable for web applications

## Matching Patterns

- Match: Simple pattern matching for message filtering
- Match Regex: Regular expression pattern matching
- Used to filter messages based on content or metadata

## Local Execution

- Enable local messaging execution
- Useful for development and testing
- Messages processed locally instead of remote
