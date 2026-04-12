# Storage Commands

## Create Storage Resource

```bash
tau new storage [flags] <name>
```

Flags:
- `--name`, `-n`: Storage resource name (can also be provided as positional argument)
- `--description`, `-d`: Storage description
- `--regex`, `-r`: Match using regex
- `--match`, `-m`: Match pattern ([^regex] if configured or /path/to/match)
- `--public`, `-p`: Make storage publicly accessible
- `--size`, `-s`: Max size either in form 10GB or 10
- `--size-unit`, `--su`: Unit if not provided with size
- `--bucket`, `-b`: Bucket type (Object, Streaming)
- `--versioning`, `-v`: Enable versioning (Object buckets)
- `--timeout`, `--ttl`: Time to live (Streaming buckets)
- `--yes`, `-y`: Skip confirmation prompts

**Interactive Prompts:**
- Description: (can be bypassed with --description)
- Use Regex For Match: True/False (can be bypassed with --regex)
- Match: (can be bypassed with --match)
- Public: (can be bypassed with --public)
- Size: (can be bypassed with --size)
- Bucket: Object/Streaming (can be bypassed with --bucket)
- Do Versioning: (can be bypassed with --versioning, Object only)
- TTL: (can be bypassed with --timeout, Streaming only)
- Confirm creation: Yes/No (can be bypassed with --yes)

Examples:
```bash
# Non-interactive mode (ALWAYS use these flags):
tau new storage --description "My storage" --bucket Object --regex false --public false --defaults --yes my-storage
tau new storage --description "Object bucket" --bucket Object --regex false --public false --defaults --yes object-bucket
tau new storage --description "Streaming bucket" --bucket Streaming --regex false --public false --defaults --yes streaming-bucket
tau new storage --description "Public storage" --bucket Object --regex false --public --defaults --yes public-storage
tau new storage --description "Versioned storage" --bucket Object --regex false --public false --versioning --defaults --yes versioned-storage

# Interactive mode (will prompt):
tau new storage
tau new storage my-storage
tau new storage --bucket Object object-bucket
tau new storage --bucket Streaming streaming-bucket
tau new storage --public public-storage
tau new storage \
  --bucket Object \
  --versioning \
  versioned-storage
```

## Query Storage Resource

```bash
tau query storage [flags] <name>
```

Flags:
- `--name`, `-n`: Storage resource name (can also be provided as positional argument)
- `--list`: List all storage resources instead of querying
- `--select`: Force selection prompt

Examples:
```bash
tau query storage
tau query storage my-storage
tau query storage --list
```

## List Storage Resources

```bash
tau list storage
```

Examples:
```bash
tau list storage
```

## Edit Storage Resource

```bash
tau edit storage <name>
```

Examples:
```bash
tau edit storage
tau edit storage my-storage
```

## Delete Storage Resource

```bash
tau delete storage <name>
```

Flags:
- `--yes`, `-y`: Skip confirmation prompts

Examples:
```bash
tau delete storage
tau delete storage my-storage
```

## Non-Interactive Mode

**CRITICAL**: For non-interactive mode, ALWAYS include ALL required flags:

```bash
tau new storage --description "[description]" --bucket Object --regex false --public false --defaults --yes <name>
```

**Required flags:**
- `--description`: Storage description (use empty string `""` if not needed)
- `--bucket`: Bucket type (`Object` or `Streaming`)
- `--regex false`: Disable regex matching (or use `--regex` for true, or `--match` for pattern)
- `--public false`: Make storage private (or use `--public` for true)
- `--yes`: Skip confirmation prompts

**Optional flags:**
- `--size`: Maximum size (e.g., `--size 10GB` or `--size 10 --size-unit GB`)
- `--versioning`: Enable versioning (Object buckets only)
- `--timeout` or `--ttl`: Time to live (Streaming buckets only)

## Common Patterns

### Object Storage Setup (Non-Interactive)
```bash
tau new storage \
  --description "Object storage bucket" \
  \
  --bucket Object \
  --regex false \
  --public false \
  --size 10GB \
  --versioning \
  --defaults --yes \
  object-bucket
```

### Streaming Storage Setup (Non-Interactive)
```bash
tau new storage \
  --description "Streaming storage bucket" \
  \
  --bucket Streaming \
  --regex false \
  --public false \
  --size 5GB \
  --ttl 1h \
  --defaults --yes \
  streaming-bucket
```

### Public Storage (Non-Interactive)
```bash
tau new storage \
  --description "Public storage bucket" \
  \
  --bucket Object \
  --regex false \
  --public \
  --size 20GB \
  --defaults --yes \
  public-bucket
```

### Storage with Matching (Non-Interactive)
```bash
tau new storage \
  --description "Filtered storage" \
  \
  --bucket Object \
  --regex false \
  --match "user-*" \
  --public false \
  --size 10GB \
  --defaults --yes \
  filtered-storage
```

### Storage with Regex Matching (Non-Interactive)
```bash
tau new storage \
  --description "Regex storage" \
  \
  --bucket Object \
  --regex \
  --match "^data-[0-9]+$" \
  --public false \
  --size 5GB \
  --defaults --yes \
  regex-storage
```

## Advanced Usage

### Large Storage (Non-Interactive)
```bash
tau new storage \
  --description "Large storage bucket" \
  \
  --bucket Object \
  --regex false \
  --public false \
  --size 100GB \
  --versioning \
  --defaults --yes \
  large-bucket
```

### Streaming with TTL (Non-Interactive)
```bash
tau new storage \
  --description "Temporary streaming storage" \
  \
  --bucket Streaming \
  --regex false \
  --public false \
  --ttl 30m \
  --size 2GB \
  --defaults --yes \
  temp-stream
```

### Versioned Object Storage (Non-Interactive)
```bash
tau new storage \
  --description "Versioned object storage" \
  \
  --bucket Object \
  --regex false \
  --public \
  --versioning \
  --size 50GB \
  --defaults --yes \
  versioned
```

### Multiple Storage Resources (Non-Interactive)
```bash
tau new storage --description "Object store" --bucket Object --regex false --public false --defaults --yes object-store
tau new storage --description "Stream store" --bucket Streaming --regex false --public false --defaults --yes stream-store
tau new storage --description "Public store" --bucket Object --regex false --public --defaults --yes public-store
```

## Troubleshooting

### Error: "Storage resource name already exists"
- Use a different name
- Or edit the existing storage resource instead
- Check existing storage: `tau list storage`

### Error: "Invalid bucket type"
- Valid types: `Object`, `Streaming`
- Check spelling and case
- Use exact type name

### Error: "Invalid size specification"
- Use format: `--size 10GB` or `--size 10 --size-unit GB`
- Valid units: MB, GB

### Error: "Versioning only for Object buckets"
- Versioning is only available for Object buckets
- Use `--bucket Object` with `--versioning`

### Error: "TTL only for Streaming buckets"
- TTL is only available for Streaming buckets
- Use `--bucket Streaming` with `--ttl`

### Error: "Invalid TTL format"
- Use duration format: `30s`, `5m`, `1h`
- Examples: `30s`, `2m30s`, `1h`
