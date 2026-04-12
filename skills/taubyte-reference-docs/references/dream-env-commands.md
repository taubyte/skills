# Dream Environment Commands

## Create Multiverse

```bash
dream new multiverse
```

Behavior:
- Creates a new Dream multiverse
- Must be created before creating universes
- Required first step for Dream environment setup

Examples:
```bash
dream new multiverse
```

## Create Universe

```bash
dream new universe [name] [flags]
```

Note: Requires multiverse to be created first with `dream new multiverse`.

See [Universe Commands](universe-commands.md) for full details.

## Inject Commands

### Push All

```bash
dream inject push-all --universe [universe-name]
```

**CRITICAL**: Always specify `--universe` flag. Universe name should be extracted from current context (via `tau current`) or provided by user.

Flags:
- `--universe`, `-u` (string, required): Target universe name

Behavior:
- Pushes all resources to Dream environment
- Triggers builds for all resources
- Used after making changes in Dream environment
- **MUST be run before `push-specific`**

Examples:
```bash
dream inject push-all --universe my-universe
dream inject push-all --universe default
```

### Push Specific Resource

```bash
dream inject push-specific --universe [universe-name] --rid [resource-id] --fn [function-name]
```

**CRITICAL**: 
- Always specify `--universe` flag
- **MUST be run AFTER `push-all` completes successfully**
- Wait for `push-all` to succeed (check exit code, verify builds) before running this command

Flags:
- `--universe`, `-u` (string, required): Target universe name
- `--rid`: **Numeric GitHub repository ID** (e.g. `1157540083`). Get from `[project]/config/websites/[name].yaml` → `source.github.id`. **Do NOT use the Taubyte resource ID (Qm...)**—that will cause "strconv.Atoi: parsing ... invalid syntax".
- `--fn`: Repository fullname (format: `[github-username]/tb_website_[name]`)
  
  **How to get --rid and --fn:**
  - Read `[project]/config/websites/[name].yaml`: use `source.github.id` for `--rid`, `source.github.fullname` for `--fn`
  - Or run `tau query website --name [name]`: use the numeric ID and Repository fullname from output
  
  **Important:** The `--fn` parameter must match the actual repository fullname (e.g., `ghir-hak/tb_website_notenow_web`), not a hardcoded namespace like `taubyte0`.

Execution Order:
1. First run: `dream inject push-all --universe [universe-name]`
2. Wait and verify success: `tau query builds`
3. Only then run: `dream inject push-specific --universe [universe-name] --rid [resource-id] --fn [function-name]`

Examples:
```bash
# Step 1: Run push-all first
dream inject push-all --universe my-universe

# Step 2: Wait and verify
tau query builds

# Step 3: Get repository fullname (choose one method)
# Method A: From config file
cat my-project/config/websites/my-website.yaml
# Extract: source.github.fullname (e.g., "ghir-hak/tb_website_my-website")

# Method B: From query
tau query website --name my-website
# Extract from Repository field: https://github.com/ghir-hak/tb_website_my-website
# Use: ghir-hak/tb_website_my-website

# Step 4: Run push-specific for EACH website (required after push-all)
# --rid = source.github.id (numeric) from website config, NOT the Taubyte resource ID (Qm...)
dream inject push-specific --universe my-universe --rid 903606675 --fn ghir-hak/tb_website_my-website
```

**Note:** Replace with your actual `source.github.id` and `source.github.fullname` from each website's config. **You MUST run push-specific for every website** after push-all; website builds are not triggered by push-all alone.

## Other Dream Commands

The Dream CLI supports:
- `dream new multiverse` - Create multiverse (required first)
- `dream new universe [name]` - Create universe (requires multiverse)
- `dream inject` (with subcommands: services, simple, service-name, set-branch, push-all, attach-plugin, push-specific)
- `dream kill` (with subcommands: universe, simple, services, service-name)
- `dream status` (with subcommands: universe, id, service-name)

See [Universe Commands](universe-commands.md) and [Multiverse Commands](multiverse-commands.md) for details.
