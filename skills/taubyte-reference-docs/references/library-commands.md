# Library Commands

## Create Library

```bash
tau new library [flags] <name>
```

Flags:
- `--name`, `-n`: Library name (can also be provided as positional argument)
- `--description`, `-d`: Library description
- `--template`: Template URL or name
- `--provider`: Git provider
- `--path`, `-p`: Repository path
- `--repository-name`, `--repo-n`: Repository name
- `--repository-id`, `--repo-id`: Repository ID
- `--clone`: Clone repository after creation
- `--embed-token`, `-e`: Embed authentication token
- `--generate-repository`, `-g`: Generate new repository
- `--private`: Create private repository
- `--branch`, `-b`: Branch name
- `--yes`, `-y`: Skip confirmation prompts

**Interactive Prompts:**
- Description: (can be bypassed with --description)
- Git Provider: (can be bypassed with --provider)
- Generate Repository: True/False (can be bypassed with --generate-repository)
- Private: True/False (can be bypassed with --private)
- Select Template: (can be bypassed with --template)
- Path: (can be bypassed with --path)
- Branch: (can be bypassed with --branch)
- Confirm creation: Yes/No (can be bypassed with --yes)
- Embed Git Token: True/False (can be bypassed with --embed-token)

**CRITICAL**: When creating libraries with `--generate-repository`, ALWAYS include `--branch main` to ensure GitHub creates repository with `main` branch, never `master`.

Examples:
```bash
tau new library
tau new library my-lib
tau new library --generate-repository --branch main my-lib
tau new library --repository-name user/my-lib-repo --branch main my-lib
```

## Clone Library

```bash
tau clone library [flags] <name>
```

Flags:
- `--name`, `-n`: Library name (can also be provided as positional argument)
- `--branch`, `-b`: Branch to checkout
- `--embed-token`, `-e`: Embed authentication token

**Interactive Prompts:**
- Embed Git Token: True/False (can be bypassed with --embed-token)
- Select Branch: (can be bypassed with --branch)

Examples:
```bash
tau clone library
tau clone library my-lib
tau clone library --branch develop my-lib
```

## Push Library

```bash
tau push library <name>
```

**Important**: Do not include trailing `/` in library name.

Flags:
- `--message`, `-m`: Commit message

**Interactive Prompts:**
- Commit Message: (can be bypassed with --message)

Examples:
```bash
tau push library
# Prompts: Select library, Commit Message
tau push library my-lib
# Prompts: Commit Message
# Correct: tau push library my-lib
# Wrong: tau push library my-lib/
```

**Note**: In remote cloud, GitHub webhook triggers build automatically. In Dream environment, use `dream inject push-all --universe [universe-name]` or `dream inject push-specific --universe [universe-name] --rid [resource-id] --fn [function-name]` to trigger builds. **CRITICAL**: Always specify `--universe` flag. Always run `push-all` first and wait for it to succeed before running `push-specific`.

## Pull Library

```bash
tau pull library <name>
```

Examples:
```bash
tau pull library
tau pull library my-lib
```

## Checkout Library Branch

```bash
tau checkout library [flags] <name>
```

Flags:
- `--name`, `-n`: Library name (can also be provided as positional argument)
- `--branch`: Branch name to checkout

Examples:
```bash
tau checkout library
tau checkout library --branch feature/new-feature my-lib
```

## Import Library

```bash
tau import library <name>
```

Examples:
```bash
tau import library
tau import library my-lib
```

## Query Library

```bash
tau query library [flags] <name>
```

Flags:
- `--name`, `-n`: Library name (can also be provided as positional argument)
- `--list`: List all libraries instead of querying
- `--select`: Force selection prompt

Examples:
```bash
tau query library
tau query library my-lib
tau query library --list
```

## List Libraries

```bash
tau list libraries
```

Aliases:
- `tau list library`

Examples:
```bash
tau list libraries
```

## Edit Library

```bash
tau edit library <name>
```

Examples:
```bash
tau edit library
tau edit library my-lib
```

## Delete Library

```bash
tau delete library <name>
```

Flags:
- `--yes`, `-y`: Skip confirmation prompts

Examples:
```bash
tau delete library
tau delete library my-lib
```

## Common Patterns

### Creating and Using Library
```bash
tau new library --generate-repository --branch main utils
tau clone library --branch main utils
tau push library utils
tau new function --source library/utils my-function
```

### Library with Template
```bash
tau new library --template template-url --branch main my-lib
```

### Library from Existing Repository
```bash
tau import library --repository-name user/my-lib-repo my-lib
```

### Branch Management
```bash
tau checkout library --branch feature/new-feature my-lib
tau push library my-lib
tau checkout library --branch main my-lib
```

## Advanced Usage

### Private Library
```bash
tau new library --private --generate-repository --branch main private-lib
```

### Library with Custom Repository
```bash
tau new library \
  --repository-name myorg/my-lib-repo \
  --provider github \
  --branch main \
  my-lib
```

### Multiple Libraries
```bash
tau new library utils
tau new library helpers
tau new library shared
tau new function --source library/utils api
tau new function --source library/helpers worker
```

## Troubleshooting

### Error: "Library name already exists"
- Use a different name
- Or edit the existing library instead
- Check existing libraries: `tau list libraries`

### Error: "Repository not found"
- Verify repository name is correct
- Check repository visibility (public/private)
- Verify authentication: `tau login`

### Error: "Failed to clone library"
```bash
tau list libraries
tau login
tau clone library my-lib
```

### Error: "Branch not found"
- List available branches
- Use valid branch name
- Create branch if needed

### Library Not Available in Functions
- Verify library exists: `tau list libraries`
- Check library is cloned locally
- Use format: `library/<library-name>`
