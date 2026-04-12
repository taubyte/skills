# Authentication Commands

## Login

```bash
tau login
```

Behavior:
- If only one profile exists, automatically logs in with that profile
- If multiple profiles exist, opens an interactive selection menu
- If no profiles exist, automatically creates a new profile

## Login with Profile

```bash
tau login <profile-name>
```

Examples:
```bash
tau login my-profile
tau login default
```

## Create New Profile

```bash
tau login --new [profile-name]
```

Flags:
- `--new`: Force creation of a new profile
- `--set-default`, `-d`: Set the newly created profile as the default
- `--token`, `-t`: Provide authentication token directly
- `--provider`, `-p`: Specify the git provider (default: "github")
- `--name`: Specify the profile name directly (or provide as argument)

**Interactive Prompts:**
- Use as default: True/False (can be bypassed with --set-default)
- Git Provider: (can be bypassed with --provider)
- Get token with: Select authentication method (cannot bypass if using web auth)

Examples:
```bash
tau login --new my-profile
tau login --new --name production-profile
tau login --new --token ghp_xxxxxxxxxxxx --provider github
tau login --new --set-default
```

**After creating profile**, use `tau login [profile-name]` to switch to it.

## Flag Reference

| Flag | Short | Description |
|------|-------|-------------|
| `--new` | - | Create a new authentication profile |
| `--set-default` | `-d` | Set the profile as the default profile |
| `--token` | `-t` | Authentication token from git provider |
| `--provider` | `-p` | Git provider (currently only "github" supported) |
| `--name` | - | Profile name (used as argument or flag) |

## Common Patterns

### Initial Setup
```bash
tau login
tau login --new --token ghp_xxxxxxxxxxxx
```

### Multiple Profiles
```bash
tau login --new --name production --token ghp_prod_token
tau login --new --name development --token ghp_dev_token
tau login production
tau login development
```

### Setting Default Profile
```bash
tau login --new --set-default --name main-profile
tau login existing-profile --set-default
```

### Quick Profile Switch
```bash
tau login profile-name
```

## Advanced Usage

### Non-Interactive Profile Creation
```bash
tau login --new \
  --name my-profile \
  --token ghp_xxxxxxxxxxxx \
  --provider github \
  --set-default
```

### Profile Management Workflow
```bash
tau login --new --name prod --token ghp_prod_token
tau login --new --name dev --token ghp_dev_token
tau login --new --name staging --token ghp_staging_token
tau login prod --set-default
tau login dev
tau login staging
tau login prod
```

## Troubleshooting

### Error: "Profile does not exist"
```bash
tau login
tau login --new --name <profile-name>
```

### Error: "Invalid token"
1. Generate a new token from your git provider
2. Create a new profile:
```bash
tau login --new --name <profile-name> --token <new-token>
```

### Error: "Failed to extract git info"
- This is a warning, not an error
- Profile will still be created
- Verify token has appropriate permissions

### No Profiles Available
```bash
tau login --new
tau login --new --token ghp_xxxxxxxxxxxx
```

### Multiple Profiles, No Default
```bash
tau login <profile-name> --set-default
tau login --new --set-default --name <profile-name>
```

### Profile Selection Issues
1. Verify profile exists in configuration
2. Check profile name spelling (case-sensitive)
3. Use explicit profile name: `tau login <profile-name>`

## Environment Variables

- `TAUBYTE_PROFILE`: Currently selected profile name
- `TAUBYTE_CONFIG`: Location of tau configuration file (default: `~/tau.yaml`)
