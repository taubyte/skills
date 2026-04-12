# Project Commands

## Create Project

```bash
tau new project [flags] <name>
```

Flags:
- `--name`, `-n`: Project name (can also be provided as positional argument)
- `--description`, `-d`: Project description
- `--location`, `--loc`: Directory location
- `--embed-token`, `-e`: Embed authentication token (use `--no-embed-token` or `--no-e` to disable)
- `--public`: Create public repositories
- `--private`: Create private repositories
- `--yes`, `-y`: Skip confirmation prompts only (does not skip boolean prompts like embed-token)

**Interactive Prompts:**
- Visibility: private/public (can be bypassed with --private/--public)
- Embed Git Token: True/False (can be bypassed with --embed-token or --no-embed-token)
- Confirm creation: Yes/No (can be bypassed with --yes)

Examples:
```bash
tau new project
tau new project --description "My awesome project" my-project
tau new project --description "My awesome project" my-project
tau new project --loc /path/to/projects my-project
tau new project --private my-project
```

**Note**: Project name should not include trailing `/` when selecting later with `tau select project`.

## Clone Project

```bash
tau clone project [flags]
```

Flags:
- `--location`, `--loc`: Directory location
- `--branch`, `-b`: Branch to checkout
- `--embed-token`, `-e`: Embed authentication token (use `--no-embed-token` or `--no-e` to disable)
- `--select`: Force selection prompt
- `--yes`, `-y`: Skip confirmation prompts only (does not skip boolean prompts like embed-token)

**Interactive Prompts:**
- Embed Git Token: True/False (can be bypassed with --embed-token or --no-embed-token)
- Select Branch: (can be bypassed with --branch)

**Note**: `tau clone project` clones only the project's config and code repositories. Websites and libraries have separate repositories and must be cloned separately using `tau clone website <name>` or `tau clone library <name>`.

Examples:
```bash
tau clone project
tau clone project --select
tau clone project --loc /path/to/clone
tau clone project --branch develop
tau clone project --no-embed-token
tau clone project --no-e --branch main
```

## Import Project

```bash
tau import project [flags]
```

Flags:
- `--config`: Config repository name (interactive if not provided)
- `--code`: Code repository name (interactive if not provided)
- `--yes`, `-y`: Skip confirmation prompts

Examples:
```bash
tau import project
tau import project --config tb_config_myproject --code tb_code_myproject
```

Note: Import looks for repositories with naming pattern `tb_<type>_<name>` where type is "config" or "code".

## Push Project

```bash
tau push project [flags]
```

Flags:
- `--message`, `-m`: Commit message
- `--config-only`, `--config`: Push only config repo
- `--code-only`, `--code`: Push only code repo

**Interactive Prompts:**
- Commit Message: (can be bypassed with --message)

Examples:
```bash
tau push project
# Prompts: Commit Message
tau push project --message "Update configuration"
tau push project --config-only
# Prompts: Commit Message
tau push project --code-only
# Prompts: Commit Message
```

## Pull Project

```bash
tau pull project [flags]
```

Flags:
- `--config-only`, `--config`: Pull only configuration repository
- `--code-only`, `--code`: Pull only code repository

**Note**: If the repository is already up-to-date, this command may show an error message. This is expected behavior and indicates no changes were pulled.

Examples:
```bash
tau pull project
tau pull project --config-only
tau pull project --code-only
```

## Checkout Project Branch

```bash
tau checkout project [flags]
```

Flags:
- `--branch`: Branch name to checkout (required in non-interactive mode)

**Note**: In non-interactive mode (when stdin is not a TTY), you must provide the `--branch` flag. Interactive prompts will not work.

Examples:
```bash
tau checkout project --branch develop
tau checkout project --branch main
```

## Query Project

```bash
tau query project <name>
```

Examples:
```bash
tau query project
tau query project my-project
```

## List Projects

```bash
tau list projects
```

Aliases:
- `tau list project`

Examples:
```bash
tau list projects
```

## Select Project

```bash
tau select project <name>
```

**Important**: Do not include trailing `/` in project name.

Examples:
```bash
tau select project
tau select project my-project
# Correct: tau select project my-project
# Wrong: tau select project my-project/
```

## Edit Project

```bash
tau edit project <name>
```

Examples:
```bash
tau edit project
tau edit project my-project
```

## Delete Project

```bash
tau delete project <name>
```

Examples:
```bash
tau delete project
tau delete project my-project
```

## Common Patterns

### Initial Project Setup
```bash
tau new project --description "My project"
tau new application
tau new function
```

### Import Existing Project
```bash
tau import project
tau import project \
  --config tb_config_myproject \
  --code tb_code_myproject
```

### Working with Multiple Projects
```bash
tau clone project --loc ~/projects/project1
tau clone project --loc ~/projects/project2
tau select project project1
tau select project project2
```

### Branch Management
```bash
tau checkout project --branch feature/new-feature
tau checkout project --branch main
tau push project --message "Add new feature"
```

### Selective Repository Operations
```bash
tau pull project --config-only
tau push project --config-only --message "Update config"
tau pull project --code-only
tau push project --code-only --message "Update code"
```

## Advanced Usage

### Project with Custom Location
```bash
tau new project --loc ~/my-projects/my-app
tau clone project --loc /opt/projects/my-app
```

### Private Project Setup
```bash
tau new project --private --description "Private project"
```

### Import Workflow
```bash
tau import project
```

### Embedding Tokens
```bash
tau new project --embed-token
tau clone project --embed-token
tau clone project --no-embed-token
tau clone project --no-e
```

## Troubleshooting

### Error: "Project not found"
```bash
tau list projects
tau select project <name>
tau clone project
```

### Error: "Repository not found"
1. Verify project exists: `tau list projects`
2. Check repository names follow pattern: `tb_config_<name>` and `tb_code_<name>`
3. Verify authentication: `tau login`
4. Import project if repositories exist: `tau import project`

### Error: "Both flags cannot be true"
- Use only one flag at a time
- Omit both flags to operate on both repositories

### Error: "Failed to clone project"
```bash
tau login
tau list projects
tau clone project --loc /path/to/clone
```

### Error: "Branch checkout failed"
```bash
tau checkout project --branch [branch-name]
```

### Import Not Finding Repositories
1. Verify repository naming: `tb_config_<name>` or `tb_code_<name>`
2. Check repository visibility (public/private)
3. Verify authentication has access
4. Repositories must be under your GitHub account or organization

### Project Selection Issues
```bash
tau list projects
tau select project <name>
tau current
```

### Error: "opening project config at ... failed" or "Project not cloned locally"
Some commands require the project to be cloned locally before they can work. These include:
- `tau list websites`
- `tau list functions`
- `tau query builds`
- `tau pull project`
- `tau push project`

**Solution:**
```bash
tau clone project --no-embed-token
# Or with specific branch:
tau clone project --no-embed-token --branch main
```

### Error: "cannot prompt: non-interactive mode"
This error occurs when running commands in non-interactive environments (scripts, CI/CD, or when stdin is not a TTY).

**Solution:**
Provide all required flags explicitly:
```bash
# Instead of: tau clone project
tau clone project --no-embed-token --branch main

# Instead of: tau checkout project
tau checkout project --branch main

# Instead of: tau new project
tau new project --description "Description" --private --no-embed-token --defaults --yes my-project
```

**Note**: The `--yes` flag only skips confirmation prompts (Yes/No questions), not boolean prompts like embed-token. Use explicit flags like `--no-embed-token` instead.

## Environment Variables

- `TAUBYTE_PROJECT`: Currently selected project name
- `TAUBYTE_CONFIG`: Location of tau configuration file (default: `~/tau.yaml`)
