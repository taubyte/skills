# SmartOps Commands

## Create SmartOp

```bash
tau new smartops [flags] <name>
```

Flags:
- `--name`, `-n`: SmartOp name (can also be provided as positional argument)
- `--description`: SmartOp description
- `--timeout`: Execution timeout duration
- `--memory`: Memory allocation
- `--memory-unit`: Memory unit (MB, GB)
- `--source`: Source code path
- `--call`: Call URL or endpoint
- `--template`: Template URL or name
- `--language`: Programming language
- `--use-code-template`: Use code template
- `--yes`, `-y`: Skip confirmation prompts

Examples:
```bash
tau new smartops
tau new smartops data-processor
tau new smartops \
  --timeout 60s \
  --memory 512MB \
  processor
tau new smartops \
  --source ./smartops/processor \
  --entry-point handler \
  my-op
```

## Query SmartOp

```bash
tau query smartops [flags] <name>
```

Flags:
- `--name`, `-n`: SmartOp name (can also be provided as positional argument)
- `--list`: List all SmartOps instead of querying
- `--select`: Force selection prompt

Examples:
```bash
tau query smartops
tau query smartops data-processor
tau query smartops --list
```

## List SmartOps

```bash
tau list smartops
```

Examples:
```bash
tau list smartops
```

## Edit SmartOp

```bash
tau edit smartops <name>
```

Examples:
```bash
tau edit smartops
tau edit smartops data-processor
```

## Delete SmartOp

```bash
tau delete smartops <name>
```

Flags:
- `--yes`, `-y`: Skip confirmation prompts

Examples:
```bash
tau delete smartops
tau delete smartops data-processor
```

## Common Patterns

### Basic SmartOp Setup
```bash
tau new smartops data-processor
```

### SmartOp with Configuration
```bash
tau new smartops \
  --timeout 60s \
  --memory 1024MB \
  --source ./smartops/processor \
  --entry-point main \
  processor
```

### SmartOp from Template
```bash
tau new smartops \
  --template template-url \
  --language javascript \
  my-op
```

### SmartOp with Call URL
```bash
tau new smartops \
  --call https://external-api.com/endpoint \
  proxy-op
```

## Advanced Usage

### SmartOp with Custom Memory
```bash
tau new smartops \
  --memory 2048 \
  --memory-unit MB \
  --timeout 5m \
  big-processor
```

### SmartOp with Source Library
```bash
tau new smartops \
  --source library/my-lib \
  lib-op
```

### Multiple SmartOps
```bash
tau new smartops processor-1
tau new smartops processor-2
tau new smartops processor-3
```

## Troubleshooting

### Error: "SmartOp name already exists"
- Use a different name
- Or edit the existing SmartOp instead
- Check existing SmartOps: `tau list smartops`

### Error: "Invalid timeout format"
- Use duration format: `30s`, `5m`, `1h`
- Examples: `30s`, `2m30s`, `1h`

### Error: "Invalid memory specification"
- Use format: `--memory 512MB` or `--memory 512 --memory-unit MB`
- Valid units: MB, GB

### Error: "Source not found"
- Verify source path is correct
- Use relative path from project root
- Or use library source: `library/<name>`
