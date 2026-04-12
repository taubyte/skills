# Domain Commands

## Create Domain

```bash
tau new domain [flags] <name>
```

Flags:
- `--name`, `-n`: Domain name (can also be provided as positional argument)
- `--description`, `-d`: Domain description
- `--generated-fqdn`, `--g-fqdn`: Generate an FQDN based on the project ID
- `--generated-fqdn-prefix`, `--g-prefix`: Prefix to use when generating an FQDN
- `--fqdn`, `-f`: Fully Qualified Domain Name
- `--cert-type`, `--type`: Certificate type
- `--certificate`, `-c`: Certificate file path
- `--key`, `-k`: Private key file path
- `--yes`, `-y`: Skip confirmation prompts

**Interactive Prompts:**
- Description: (can be bypassed with --description)
- Generate FQDN: True/False (can be bypassed with --generated-fqdn)
- FQDN prefix: (can be bypassed with --generated-fqdn-prefix)
- Certificate type: (can be bypassed with --cert-type)
- Confirm creation: Yes/No (can be bypassed with --yes)

**Domain Selection**: When selecting domains for resources (e.g., websites), use arrow keys to navigate and right/left arrows to check/uncheck domains.

Examples:
```bash
# Non-interactive mode (ALWAYS use these flags):
tau new domain --description "My API domain" --generated-fqdn --defaults --yes my-api
tau new domain --description "API domain" --fqdn api.example.com --defaults --yes api.example.com

# Interactive mode (will prompt):
tau new domain
tau new domain --fqdn api.example.com api.example.com
tau new domain my-api
# Prompts: Description, Generate FQDN, FQDN prefix, Certificate type
```

## Query Domain

```bash
tau query domain [flags] <name>
```

Flags:
- `--name`, `-n`: Domain name (can also be provided as positional argument)
- `--list`: List all domains instead of querying
- `--select`: Force selection prompt

Examples:
```bash
tau query domain
tau query domain api.example.com
tau query domain --list
```

## List Domains

```bash
tau list domains
```

Aliases:
- `tau list domain`

Examples:
```bash
tau list domains
```

## Edit Domain

```bash
tau edit domain [flags] <name>
```

Flags:
- `--description`, `-d`: Domain description
- `--fqdn`, `-f`: FQDN
- `--cert-type`, `--type`: Certificate type
- `--certificate`, `-c`: Certificate file path
- `--key`, `-k`: Private key file path
- `--yes`, `-y`: Skip confirmation prompts

**Interactive Prompts:**
- Description: (can be bypassed with --description)
- FQDN: (can be bypassed with --fqdn)
- Certificate type: (can be bypassed with --cert-type)
- Confirm edit: Yes/No (can be bypassed with --yes)

Examples:
```bash
tau edit domain
tau edit domain api.example.com
```

## Delete Domain

```bash
tau delete domain <name>
```

Flags:
- `--yes`, `-y`: Skip confirmation prompts

Examples:
```bash
tau delete domain
tau delete domain api.example.com
```

## Non-Interactive Mode

**CRITICAL**: For non-interactive mode, ALWAYS include ALL required flags:

```bash
tau new domain --description "[description]" --generated-fqdn --defaults --yes <name>
```

**Required flags:**
- `--description`: Domain description (use empty string `""` if not needed)
- `--generated-fqdn`: Generate FQDN automatically (or use `--fqdn` for custom)
- `--yes`: Skip confirmation prompts

## Common Patterns

### Basic Domain Setup (Non-Interactive)
```bash
tau new domain --description "API domain" --fqdn api.example.com --defaults --yes api.example.com
```

### Generated Domain for Development (Non-Interactive)
```bash
tau new domain --description "Development API domain" --generated-fqdn --defaults --yes dev-api
```

### Domain with Certificate (Non-Interactive)
```bash
tau new domain \
  --description "Secure domain" \
  --fqdn secure.example.com \
  --cert-type custom \
  --certificate /path/to/cert.pem \
  --key /path/to/key.pem \
  --yes \
  secure.example.com
```

### Multiple Domains (Non-Interactive)
```bash
tau new domain --description "API domain" --fqdn api.example.com --defaults --yes api.example.com
tau new domain --description "WWW domain" --fqdn www.example.com --defaults --yes www.example.com
tau new domain --description "Admin domain" --fqdn admin.example.com --defaults --yes admin.example.com
```

## Advanced Usage

### Domain for Different Networks (Non-Interactive)
```bash
tau new domain --description "Dream API domain" --generated-fqdn --defaults --yes dream-api
tau new domain --description "Remote API domain" --generated-fqdn --defaults --yes remote-api
```

## Troubleshooting

### Error: "Domain name already exists"
- Use a different name
- Or edit the existing domain instead
- Check existing domains: `tau list domains`

### Error: "Invalid FQDN format"
- Use proper FQDN format: `host.domain.tld`
- Example: `api.example.com`
- Avoid trailing dots or invalid characters

### Error: "Domain validation failed"
- Verify domain is accessible
- Check network connectivity
- Use interactive prompt to generate FQDN: Select "Generate FQDN: True"

### Error: "Certificate file not found"
- Verify file paths are correct
- Use absolute paths if needed
- Check file permissions

### Generated Domain Not Working
- Generated domains work within the network context
- Dream domains work in local universe
- Remote domains require network configuration
- Verify network selection: `tau current`
