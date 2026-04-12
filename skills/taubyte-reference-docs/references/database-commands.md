# Database Commands

## Create Database

```bash
tau new database [flags] <name>
```

Flags:
- `--name`, `-n`: Database name (can also be provided as positional argument)
- `--description`, `-d`: Database description
- `--regex`, `-r`: Match using regex
- `--match`, `-m`: Match pattern ([^regex] if configured or /path/to/match)
- `--local`, `-l`: Enable local execution
- `--encryption`: Enable encryption
- `--encryption-key`, `-k`, `--key`: Encryption key
- `--min`: Minimum replicas to keep (never use 0; use 1 or more)
- `--max`: Maximum replicas to keep
- `--size`, `-s`: Max size either in form 10GB or 10
- `--size-unit`, `--su`: Unit if not provided with size
- `--yes`, `-y`: Skip confirmation prompts

**Interactive Prompts:**
- Description: (can be bypassed with --description)
- Use Regex For Match: True/False (can be bypassed with --regex)
- Match: (can be bypassed with --match)
- Local: (can be bypassed with --local)
- Encrypt Database: (can be bypassed with --encryption)
- Minimum Replicas: (can be bypassed with --min)
- Maximum Replicas: (can be bypassed with --max)
- Size: (can be bypassed with --size)
- Confirm creation: Yes/No (can be bypassed with --yes)

Examples:
```bash
# Non-interactive mode (ALWAYS use these flags, including --min, --max, --size):
tau new database --description "User database" --regex false --local false --encryption false --min 1 --max 1 --size 1GB --defaults --yes user-db
tau new database --description "User database" --regex false --local false --encryption false --min 1 --max 1 --size 10GB --defaults --yes user-db
tau new database --description "Secure database" --regex false --local false --encryption --encryption-key my-key --min 1 --max 1 --size 1GB --defaults --yes secure-db
tau new database --description "Small database" --regex false --local false --encryption false --min 1 --max 1 --size 512 --size-unit MB --defaults --yes small-db

# Interactive mode (will prompt):
tau new database
tau new database user-db
tau new database --size 10GB user-db
tau new database --encryption --encryption-key my-key secure-db
tau new database --size 512 --size-unit MB small-db
```

## Query Database

```bash
tau query database [flags] <name>
```

Flags:
- `--name`, `-n`: Database name (can also be provided as positional argument)
- `--list`: List all databases instead of querying
- `--select`: Force selection prompt

Examples:
```bash
tau query database
tau query database user-db
tau query database --list
```

## List Databases

```bash
tau list databases
```

Aliases:
- `tau list database`

Examples:
```bash
tau list databases
```

## Edit Database

```bash
tau edit database <name>
```

Examples:
```bash
tau edit database
tau edit database user-db
```

## Delete Database

```bash
tau delete database <name>
```

Flags:
- `--yes`, `-y`: Skip confirmation prompts

Examples:
```bash
tau delete database
tau delete database user-db
```

## Non-Interactive Mode

**CRITICAL**: For non-interactive mode, ALWAYS include ALL required flags including `--min`, `--max`, and `--size` (with `--defaults` these are required, not optional):

```bash
tau new database --description "[description]" --regex false --local false --encryption false --min 1 --max 1 --size 1GB --defaults --yes <name>
```

**Required flags:**
- `--description`: Database description (use empty string `""` if not needed)
- `--regex false`: Disable regex matching (or use `--regex` for true, or `--match` for pattern)
- `--local false`: Disable local execution (or use `--local` for true)
- `--encryption false`: Disable encryption (or use `--encryption` with `--encryption-key` for true)
- `--min`: Minimum replicas (e.g. `--min 1`)
- `--max`: Maximum replicas (e.g. `--max 1`)
- `--size`: Maximum size (e.g. `--size 1GB` or `--size 10 --size-unit GB`)
- `--yes`: Skip confirmation prompts

**Optional flags:**
- `--match`: Match pattern (if not using regex)

## Common Patterns

### Basic Database Setup (Non-Interactive)
```bash
tau new database --description "My database" --regex false --local false --encryption false --min 1 --max 1 --size 5GB --defaults --yes my-db
```

### Encrypted Database (Non-Interactive)
```bash
tau new database \
  --description "Secure database" \
  --regex false \
  --local false \
  --encryption \
  --encryption-key my-secret-key \
  --min 1 \
  --max 1 \
  --size 10GB \
  --defaults --yes \
  secure-db
```

### Database with Matching (Non-Interactive)
```bash
tau new database \
  --description "Filtered database" \
  --regex false \
  --match "user-*" \
  --local false \
  --encryption false \
  --min 1 \
  --max 1 \
  --size 2GB \
  --defaults --yes \
  filtered-db
```

### Database with Range Matching (Non-Interactive)
```bash
tau new database \
  --description "Range database" \
  \
  --regex false \
  --local false \
  --encryption false \
  --min 1 \
  --max 100 \
  --size 1GB \
  --defaults --yes \
  range-db
```

## Advanced Usage

### Large Database (Non-Interactive)
```bash
tau new database --description "Large database" --regex false --local false --encryption false --min 1 --max 1 --size 100GB --defaults --yes big-db
```

### Database with Regex Matching (Non-Interactive)
```bash
tau new database \
  --description "Regex database" \
  --regex \
  --match "^user-[0-9]+$" \
  --local false \
  --encryption false \
  --min 1 \
  --max 1 \
  --size 5GB \
  --defaults --yes \
  regex-db
```

### Local Development Database (Non-Interactive)
```bash
tau new database \
  --description "Development database" \
  --regex false \
  --local \
  --encryption false \
  --min 1 \
  --max 1 \
  --size 1GB \
  --defaults --yes \
  dev-db
```

## Troubleshooting

### Error: "Database name already exists"
- Use a different name
- Or edit the existing database instead
- Check existing databases: `tau list databases`

### Error: "Invalid size specification"
- Use format: `--size 10GB` or `--size 10 --size-unit GB`
- Valid units: MB, GB

### Error: "Encryption key required"
```bash
tau new database --description "Secure database" --regex false --local false --encryption --encryption-key my-key --min 1 --max 1 --size 1GB --defaults --yes secure-db
```
