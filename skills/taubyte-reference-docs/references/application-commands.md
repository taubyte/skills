# Application Commands

## Create Application

```bash
tau new application [flags] <name>
```

Flags:
- `--name`, `-n`: Application name (can also be provided as positional argument)
- `--description`, `-d`: Application description
- `--yes`, `-y`: Skip confirmation prompts

**Interactive Prompts:**
- Description: (can be bypassed with --description)
- Confirm creation: Yes/No (can be bypassed with --yes)

Examples:
```bash
tau new application
tau new application my-app
tau new application --description "My awesome application" my-app
tau new application --defaults --yes my-app
```

## Query Application

```bash
tau query application [flags] <name>
```

Flags:
- `--name`, `-n`: Application name (can also be provided as positional argument)
- `--list`: List all applications instead of querying
- `--select`: Force selection prompt

Examples:
```bash
tau query application
tau query application my-app
tau query application --list
```

## List Applications

```bash
tau list applications
```

Aliases:
- `tau list application`

Examples:
```bash
tau list applications
```

## Select Application

```bash
tau select application <name>
```

Examples:
```bash
tau select application
tau select application my-app
```

## Edit Application

```bash
tau edit application <name>
```

Examples:
```bash
tau edit application
tau edit application my-app
```

## Delete Application

```bash
tau delete application <name>
```

Flags:
- `--yes`, `-y`: Skip confirmation prompts

Examples:
```bash
tau delete application
tau delete application my-app
tau delete application --yes my-app
```

## Common Patterns

### Initial Application Setup
```bash
tau new application my-app
tau new function
tau new database
tau new domain
```

### Working with Multiple Applications
```bash
tau new application frontend
tau new application backend
tau new application api
tau select application frontend
tau select application backend
```

### Application Workflow
```bash
tau new application my-service
tau new function api-handler
tau new database user-db
tau new domain api.example.com
tau query application my-service
tau edit application my-service
```

### Listing and Querying
```bash
tau list applications
tau query application my-app
tau query application --select
```

## Advanced Usage

### Application with Description
```bash
tau new application \
  --description "Main e-commerce application with user management and product catalog" \
  e-commerce-platform
```

### Application Resource Organization
```bash
tau new application api
tau new function user-api
tau new function product-api
tau new database main-db
tau new domain api.example.com
```

### Querying Application Resources
```bash
tau query application my-app
```

## Troubleshooting

### Error: "No application selected"
```bash
tau select application <name>
tau new application <name>
```

### Error: "Application not found"
```bash
tau list applications
tau select application <name>
tau new application <name>
```

### Error: "Application already exists"
- Use a different name
- Or edit the existing application instead

### Application Selection Issues
```bash
tau list applications
tau select application <name>
tau current
```

### Resources Not Showing in Application
```bash
tau current
tau query application <name>
tau list functions
tau list databases
```

## Environment Variables

- `TAUBYTE_APPLICATION`: Currently selected application name
