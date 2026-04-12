# Storage Logic

## Bucket Types

### Object Buckets

- Store objects/files
- Support versioning
- Public/private access control
- Size limits
- Matching patterns

### Streaming Buckets

- Stream data
- TTL (Time To Live) configuration
- Public/private access control
- Size limits
- Matching patterns

## Storage Configuration

### Size Limits

- Maximum storage size
- Format: `--size 10GB` or `--size 10 --size-unit GB`
- Units: MB, GB

### Public Access

- Make storage publicly accessible
- Enable with `--public` flag
- Default is private

### Versioning (Object Buckets)

- Enable object versioning
- Track multiple versions of objects
- Enable with `--versioning` flag

### TTL (Streaming Buckets)

- Time to live for streaming data
- Format: duration string (e.g., "30s", "5m", "1h")
- Configure with `--ttl` flag

### Matching Patterns

- Match: Simple pattern matching
- Match Regex: Regular expression pattern matching
- Used to filter stored objects/data

### Code Usage (SDK)

- In Go code, use the **match** value (matcher) from the storage config when calling `storage.Get(match)` or `storage.New(match)`.
- Do NOT use the resource name (config filename). Example: config `my_storage.yaml` with `match: uploads` → use `storage.Get("uploads")`.
