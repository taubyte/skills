# CLI Utilities Commands

## Autocomplete

```bash
tau autocomplete
```

Usage:
- Outputs bash autocomplete script
- Should be evaluated or added to `.bashrc`
- Enables tab completion for commands and flags

Examples:
```bash
eval $(tau autocomplete)
echo 'eval $(tau autocomplete)' >> ~/.bashrc
source ~/.bashrc
echo 'eval $(tau autocomplete)' >> ~/.bash_profile
```

## Version

```bash
tau version
```

Output:
- Displays Tau CLI version
- Shows current version number

Examples:
```bash
tau version
```

## Exit

```bash
tau exit
```

Aliases:
- `tau tau` (same command)

Behavior:
- Clears current session
- Removes session file
- Resets context

Examples:
```bash
tau exit
tau tau
```

## Global Flags

### Environment Flag

```bash
tau [command] --env
```

Environment Variable:
- `TAU_ENV`: Set to enable environment mode

Usage:
- Enables environment-specific behavior
- Can be set via flag or environment variable

Examples:
```bash
tau login --env
export TAU_ENV=true
tau login
```

### Color Flag

```bash
tau [command] --color <option>
```

Options:
- `auto`: Automatic color detection (default)
- `never`: Disable color output

Environment Variable:
- `TAU_COLOR`: Set color option

Examples:
```bash
tau login --color never
tau login --color auto
export TAU_COLOR=never
tau login
```

## Common Patterns

### Autocomplete Setup
```bash
echo 'eval $(tau autocomplete)' >> ~/.bashrc
source ~/.bashrc
tau <TAB>
```

### Version Check
```bash
tau version
```

### Session Management
```bash
tau exit
tau login
tau select project
```

### Color Control
```bash
tau --color never login
tau --color auto login
```

## Advanced Usage

### Autocomplete for Different Shells
```bash
eval $(tau autocomplete)
```

### Environment-Specific Configuration
```bash
export TAU_ENV=true
export TAU_COLOR=auto
export TAU_ENV=true
export TAU_COLOR=never
```

### Script-Friendly Mode
```bash
tau --color never --env login
```

### Session Reset Workflow
```bash
tau exit
tau current
tau login
tau select project
```

## Troubleshooting

### Autocomplete Not Working
```bash
eval $(tau autocomplete)
grep "tau autocomplete" ~/.bashrc
source ~/.bashrc
```

### Version Command Issues
- Verify CLI is installed correctly
- Check binary is in PATH
- Try: `which tau`

### Exit Command Not Clearing Session
```bash
rm /tmp/tau-*
echo $TAUBYTE_SESSION
```

### Color Flag Not Working
```bash
echo $TAU_COLOR
tau --color never <command>
tau <command> --help
```

### Global Flags Not Applying
- Global flags must come before command
- Example: `tau --color never login` (not `tau login --color never`)
- Check flag syntax

## Environment Variables

- `TAU_ENV`: Enable environment mode
- `TAU_COLOR`: Color output control (auto, never)
- `TAUBYTE_PROFILE`: Selected profile
- `TAUBYTE_PROJECT`: Selected project
- `TAUBYTE_APPLICATION`: Selected application
- `TAUBYTE_CONFIG`: Config file location (default: `~/tau.yaml`)
- `TAUBYTE_SESSION`: Session file location (default: `/tmp/tau-<shell-pid>`)
