# Messaging Commands

## Create Messaging Resource

```bash
tau new messaging [flags] <name>
```

Flags:
- `--name`, `-n`: Messaging resource name (can also be provided as positional argument)
- `--description`, `-d`: Resource description
- `--local`, `-l`: Enable local execution
- `--regex`, `-r`: Match using regex
- `--match`, `-m`: Match pattern ([^regex] if configured or /path/to/match)
- `--mqtt`: Enable MQTT protocol
- `--web-socket`, `--ws`: Enable joining the pubsub through websocket
- `--yes`, `-y`: Skip confirmation prompts

**Interactive Prompts:**
- Description: (can be bypassed with --description)
- Local: (can be bypassed with --local)
- Use Regex For Match: True/False (can be bypassed with --regex)
- Channel Match: (can be bypassed with --match)
- Enable MQTT bridge: (can be bypassed with --mqtt)
- Enable WebSocket bridge: (can be bypassed with --web-socket)
- Confirm creation: Yes/No (can be bypassed with --yes)

Examples:
```bash
# Non-interactive mode (ALWAYS use these flags):
tau new messaging --description "Events messaging" --local false --regex false --mqtt false --web-socket false --defaults --yes events
tau new messaging --description "MQTT events" --local false --regex false --mqtt --web-socket false --defaults --yes mqtt-events
tau new messaging --description "WebSocket events" --local false --regex false --mqtt false --web-socket --defaults --yes ws-events
tau new messaging --description "Local events" --local --regex false --mqtt false --web-socket false --defaults --yes local-events

# Interactive mode (will prompt):
tau new messaging
tau new messaging events
tau new messaging --mqtt mqtt-events
tau new messaging --web-socket ws-events
tau new messaging --local local-events
```

## Query Messaging Resource

```bash
tau query messaging [flags] <name>
```

Flags:
- `--name`, `-n`: Messaging resource name (can also be provided as positional argument)
- `--list`: List all messaging resources instead of querying
- `--select`: Force selection prompt

Examples:
```bash
tau query messaging
tau query messaging events
tau query messaging --list
```

## List Messaging Resources

```bash
tau list messaging
```

Examples:
```bash
tau list messaging
```

## Edit Messaging Resource

```bash
tau edit messaging <name>
```

Examples:
```bash
tau edit messaging
tau edit messaging events
```

## Delete Messaging Resource

```bash
tau delete messaging <name>
```

Flags:
- `--yes`, `-y`: Skip confirmation prompts

Examples:
```bash
tau delete messaging
tau delete messaging events
```

## Non-Interactive Mode

**CRITICAL**: For non-interactive mode, ALWAYS include ALL required flags:

```bash
tau new messaging --description "[description]" --local false --regex false --mqtt false --web-socket false --defaults --yes <name>
```

**Required flags:**
- `--description`: Messaging description (use empty string `""` if not needed)
- `--local false`: Disable local execution (or use `--local` for true)
- `--regex false`: Disable regex matching (or use `--regex` for true, or `--match` for pattern)
- `--mqtt false`: Disable MQTT bridge (or use `--mqtt` for true)
- `--web-socket false`: Disable WebSocket bridge (or use `--web-socket` for true)
- `--yes`: Skip confirmation prompts

## Common Patterns

### Basic Messaging Setup (Non-Interactive)
```bash
tau new messaging --description "Events messaging" --local false --regex false --mqtt false --web-socket false --defaults --yes events
```

### MQTT Messaging (Non-Interactive)
```bash
tau new messaging \
  --description "MQTT event messaging" \
  \
  --local false \
  --regex false \
  --mqtt \
  --web-socket false \
  --defaults --yes \
  mqtt-events
```

### WebSocket Messaging (Non-Interactive)
```bash
tau new messaging \
  --description "WebSocket event messaging" \
  \
  --local false \
  --regex false \
  --mqtt false \
  --web-socket \
  --defaults --yes \
  ws-events
```

### Messaging with Matching (Non-Interactive)
```bash
tau new messaging \
  --description "Filtered events messaging" \
  \
  --local false \
  --regex false \
  --match "event-*" \
  --mqtt false \
  --web-socket false \
  --defaults --yes \
  filtered-events
```

### Local Development Messaging (Non-Interactive)
```bash
tau new messaging \
  --description "Development events messaging" \
  \
  --local \
  --regex false \
  --mqtt false \
  --web-socket false \
  --defaults --yes \
  dev-events
```

## Advanced Usage

### Multiple Protocols (Non-Interactive)
```bash
tau new messaging --description "Multi-protocol messaging" --local false --regex false --mqtt --web-socket --defaults --yes multi-protocol
tau edit messaging multi-protocol
```

### Regex Pattern Matching (Non-Interactive)
```bash
tau new messaging \
  --description "Regex events messaging" \
  \
  --local false \
  --regex \
  --match "^event-[0-9]+$" \
  --mqtt false \
  --web-socket false \
  --defaults --yes \
  regex-events
```

### Messaging for PubSub Functions (Non-Interactive)
```bash
tau new messaging --description "User events messaging" --local false --regex false --mqtt false --web-socket false --defaults --yes user-events
tau new function --description "Event handler function" --type pubsub --channel user-events --defaults --yes event-handler
```

## Troubleshooting

### Error: "Messaging resource name already exists"
- Use a different name
- Or edit the existing resource instead
- Check existing resources: `tau list messaging`

### Error: "Invalid match pattern"
- Use simple pattern: `--match "pattern-*"`
- Or regex: `--regex "^pattern-[0-9]+$"`
- Verify pattern syntax

### Messaging Not Working
```bash
tau query messaging <name>
tau query function <name>
tau query builds
```
