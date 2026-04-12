# Network Commands

## Select Network

```bash
tau select network [flags]
```

Flags:
- `--fqdn`: Remote network FQDN
- `--universe`: Dream universe name

Behavior:
- If no flags provided, opens interactive selection
- Options include: Remote, Dream, and network history
- Validates FQDN for remote networks
- Checks Dream universe availability

Examples:
```bash
tau select network
tau select network --fqdn seer.tau.example.com
tau select network --universe my-universe
```

## Common Patterns

### Selecting Remote Network
```bash
tau select network
tau select network --fqdn seer.tau.example.com
```

### Selecting Dream Network
```bash
tau select network
tau select network --universe default
```

### Switching Between Networks
```bash
tau select network --fqdn seer.tau.production.com
tau select network --universe dev
tau select network --fqdn seer.tau.production.com
```

### Network History Usage
```bash
tau select network --fqdn seer.tau.example.com
tau select network
```

## Advanced Usage

### Custom Network Configuration
```bash
tau select network --fqdn seer.tau.custom-domain.com
```

### Dream Universe Selection
```bash
tau select network --universe my-universe
```

### Multiple Network Profiles
```bash
tau login profile1
tau select network --fqdn seer.tau.prod.com
tau login profile2
tau select network --fqdn seer.tau.dev.com
```

## Troubleshooting

### Error: "Flag error"
- Use only one flag at a time
- Or omit both for interactive selection

### Error: "Universe not found"
```bash
tau select network --universe <valid-universe>
```

### Error: "FQDN validation failed"
- Verify FQDN format: `seer.tau.<domain>`
- Check network is accessible
- Verify network connectivity
- Ensure Seer is running on network

### Error: "Dream not available"
- Dream must be running for network option to appear
- Check Dream status

### Network Selection Issues
```bash
tau login
tau select network
tau current
```

### FQDN Format Issues
- Use format: `seer.tau.<domain>`
- Example: `seer.tau.example.com`
- Avoid trailing slashes or protocols

## Environment Variables

- `TAUBYTE_NETWORK`: Network type (dream, remote)
- Network URL stored in profile configuration
