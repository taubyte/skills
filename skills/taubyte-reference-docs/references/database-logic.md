# Database Logic

## Database Configuration

### Size Limits

- Maximum storage size for the database
- Can be specified with unit: `--size 10GB`
- Or separately: `--size 10 --size-unit GB`
- Units: MB, GB

### Encryption

- Optional encryption for database data
- Requires encryption key
- Currently marked as not implemented in flags

### Matching

- Match: Simple match pattern
- Match Regex: Regular expression pattern
- Min/Max: Range matching for numeric values

### Local Execution

- Enable local database execution
- Useful for development and testing

### Code Usage (SDK)

- In Go code, use the **match** value (matcher) from the database config when calling `database.New(match)`.
- Do NOT use the resource name (config filename). Example: config `noting14_db.yaml` with `match: notes` → use `database.New("notes")`.
- Use path-style keys (e.g. `user/123`, `order/789`), not colon-style (`user:123`).