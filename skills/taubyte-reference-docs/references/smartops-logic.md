# SmartOps Logic

## SmartOp Configuration

### Timeout
- Maximum execution time
- Format: duration string (e.g., "30s", "5m", "1h")
- Default varies by configuration

### Memory
- Memory allocation for execution
- Can be specified with unit: `--memory 512MB`
- Or separately: `--memory 512 --memory-unit MB`
- Units: MB, GB

### Source Code
- Path to SmartOp source code
- Entry point specifies the function to execute
- Supports various languages

### Templates
- Code templates for SmartOps
- Language-specific templates
- Use `--template` to specify template
- Use `--use-code-template` for code templates

## SmartOp Execution

- Triggered by configured events
- Automated processing
- Can call external endpoints
- Supports various languages
