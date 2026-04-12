# Service Commands

## Create Service Resource

```bash
tau new service [flags] <name>
```

Flags:
- `--name`, `-n`: Service resource name (can also be provided as positional argument)
- `--description`: Service description
- `--protocol`: Service protocol
- `--yes`, `-y`: Skip confirmation prompts

Examples:
```bash
tau new service
tau new service my-service
tau new service --protocol http api-service
```

## Query Service Resource

```bash
tau query service [flags] <name>
```

Flags:
- `--name`, `-n`: Service resource name (can also be provided as positional argument)
- `--list`: List all service resources instead of querying
- `--select`: Force selection prompt

Examples:
```bash
tau query service
tau query service my-service
tau query service --list
```

## List Service Resources

```bash
tau list services
```

Aliases:
- `tau list service`

Examples:
```bash
tau list services
```

## Edit Service Resource

```bash
tau edit service <name>
```

Examples:
```bash
tau edit service
tau edit service my-service
```

## Delete Service Resource

```bash
tau delete service <name>
```

Flags:
- `--yes`, `-y`: Skip confirmation prompts

Examples:
```bash
tau delete service
tau delete service my-service
```

## Common Patterns

### Basic Service Setup
```bash
tau new service --protocol http api-service
```

### Service in Application
```bash
tau select application my-app
tau new service --protocol http app-service
```

### Multiple Services
```bash
tau new service --protocol http api-service
tau new service --protocol p2p worker-service
tau new service --protocol pubsub event-service
```

## Advanced Usage

### Service with Custom Protocol
```bash
tau new service --protocol custom-protocol custom-service
```

### Service Configuration
```bash
tau new service my-service
tau edit service my-service
```

## Troubleshooting

### Error: "Service resource name already exists"
- Use a different name
- Or edit the existing service resource instead
- Check existing services: `tau list services`

### Error: "Invalid protocol"
- Verify protocol name is correct
- Check available protocols
- Use valid protocol identifier

### Confusion with Dream Services
- Service Resources: `tau new service` - configuration resources
- Dream Services: `dream inject service` - runtime services
- These are separate concepts
