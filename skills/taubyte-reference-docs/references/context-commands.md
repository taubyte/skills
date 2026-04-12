# Context Commands

## View Current Context

```bash
tau current
```

Aliases:
- `tau cur`
- `tau here`
- `tau this`

Output:
Displays a table with:
- Profile: Currently selected authentication profile
- Project: Currently selected project
- Application: Currently selected application (if any)
- Cloud Type: Cloud type (dream, remote, or none)
- Cloud: Cloud FQDN or universe name

## Select Cloud

```bash
tau select cloud [flags]
```

Flags:
- `--fqdn`, `-f`: FQDN of remote network to connect to
- `--universe`, `-u`: Dream universe to connect to

**Interactive Selection:**
1. Select cloud type: "remote" or "dream"
2. If "remote": Enter FQDN (e.g., `scaliny.com`)
3. If "dream": Select universe name from list

Examples:
```bash
tau select cloud
# Prompts: Select "remote" or "dream"
# If remote: Enter FQDN
# If dream: Select universe
tau select cloud --fqdn scaliny.com
tau select cloud --universe my-universe
```

Examples:
```bash
tau current
tau cur
tau here
tau this
```

## Common Patterns

### Checking Context Before Operations
```bash
tau current
```

### Context Switching Workflow
```bash
tau current
tau login different-profile
tau select project --name different-project
tau select application --name different-app
tau current
```

### Context Verification
```bash
tau current
```

## Advanced Usage

### Context in Scripts
```bash
#!/bin/bash
tau current
if [ -z "$(tau current | grep Project | awk '{print $2}')" ]; then
  echo "No project selected"
  exit 1
fi
```

### Context Troubleshooting
```bash
tau current
```

## Troubleshooting

### Context Shows "(none)"
```bash
tau login [profile-name]
tau select project --name [name]
tau new project --name [name]
tau select application --name [name]
tau new application --name [name]
tau select network
```

### Wrong Context
```bash
tau current
tau select project --name [name]
tau select application --name [name]
```

### Context Not Persisting
- Context is stored in session file
- Session file location: `/tmp/tau-<shell-pid>`
- Check session file permissions
- Verify `TAUBYTE_SESSION` environment variable

### Context Out of Sync
```bash
tau current
tau select project --name [name]
tau select application --name [name]
```

## Environment Variables

- `TAUBYTE_PROFILE`: Selected profile name
- `TAUBYTE_PROJECT`: Selected project name
- `TAUBYTE_APPLICATION`: Selected application name
- `TAUBYTE_NETWORK`: Network type
- `TAUBYTE_CONFIG`: Config file location (default: `~/tau.yaml`)
- `TAUBYTE_SESSION`: Session file location (default: `/tmp/tau-<shell-pid>`)
