# Website Commands

## Create Website

```bash
tau new website [flags] <name>
```

Flags:

- `--name`, `-n`: Website name (can also be provided as positional argument)
- `--description`, `-d`: Website description
- `--template`: Template URL or name
- `--domains`: List of domains (comma separated) by name or FQDN
- `--paths`, `-p`: HTTP paths to use for the endpoint. **CRITICAL (Windows)**: On Windows (Git Bash), prefix the command with `MSYS_NO_PATHCONV=1` to prevent path conversion: `MSYS_NO_PATHCONV=1 tau new website --paths / ...`
- `--provider`: Git provider
- `--repository-name`, `--repo-n`: Repository name
- `--repository-id`, `--repo-id`: Repository ID
- `--branch`, `-b`: Branch name
- `--clone`: Clone repository after creation
- `--embed-token`, `-e`: Embed authentication token (use `--no-embed-token` or `--no-e` to disable)
- `--generate-repository`, `-g`: Generate new repository
- `--private`: Create private repository
- `--yes`, `-y`: Skip confirmation prompts only (does not skip boolean prompts like embed-token)

**CRITICAL**: NEVER use `--public` for websites (flag is not defined). Use `--no-private` for public repos or `--private` for private repos.

**Interactive Prompts:**

- Description: (can be bypassed with --description)
- Git Provider: (can be bypassed with --provider)
- Domains: (can be bypassed with --domains, but need arrows to check/uncheck)
- Paths: (can be bypassed with --paths)
- Generate Repository: True/False (can be bypassed with --generate-repository)
- Private: True/False (can be bypassed with --private)
- Select Template: (can be bypassed with --template)
- Branch: (can be bypassed with --branch)
- Confirm creation: Yes/No (can be bypassed with --yes)
- Embed Git Token: True/False (can be bypassed with --embed-token or --no-embed-token)

**CRITICAL**: When creating websites with `--generate-repository`, ALWAYS include `--branch main` to ensure GitHub creates repository with `main` branch, never `master`.

Examples:

```bash
tau new website
tau new website my-site
# Prompts: Description, Tags, Git Provider, Domains, Paths, Generate Repository, Private, Template, Branch, Confirm, Embed Token
# Note: In non-interactive mode, name comes last: tau new website [flags] my-site
```

## Clone Website

```bash
tau clone website [flags] <name>
```

Flags:

- `--name`, `-n`: Website name (can also be provided as positional argument)
- `--branch`, `-b`: Branch to checkout
- `--embed-token`, `-e`: Embed authentication token (use `--no-embed-token` or `--no-e` to disable)

**Important**:

- Do not include trailing `/` in website name.
- Websites have separate repositories from projects. `tau clone project` only clones the project's config and code repositories, not website repositories. You must use `tau clone website <name>` to clone a website's repository.

**Interactive Prompts:**

- Embed Git Token: True/False (can be bypassed with --embed-token or --no-embed-token)
- Select Branch: (can be bypassed with --branch)

Examples:

```bash
tau clone website
# Prompts: Select website, Embed Token, Select Branch
tau clone website my-site
# Prompts: Embed Token, Select Branch
tau clone website --branch develop my-site
# Prompts: Embed Token
tau clone website --no-embed-token --branch main my-site
# Correct: tau clone website my-site
# Wrong: tau clone website my-site/
```

## Push Website

```bash
tau push website <name>
```

**Important**: Do not include trailing `/` in website name.

Flags:

- `--message`, `-m`: Commit message

**Interactive Prompts:**

- Commit Message: (can be bypassed with --message)

Examples:

```bash
tau push website
# Prompts: Select website, Commit Message
tau push website my-site
# Prompts: Commit Message
# Correct: tau push website my-site
# Wrong: tau push website my-site/
```

**Note**: In remote cloud, GitHub webhook triggers build automatically. In Dream environment, use `dream inject push-all --universe [universe-name]` or `dream inject push-specific --universe [universe-name] --rid [resource-id] --fn [function-name]` to trigger builds. **CRITICAL**: Always specify `--universe` flag. Always run `push-all` first and wait for it to succeed before running `push-specific`.

## Website repository: `.taubyte/build.sh`

**CRITICAL:** The website build script must **never be empty**. The pipeline serves only what is placed in `/out`. If `build.sh` does nothing, the site will not deploy.

- **After cloning or creating a website** (especially with `--template empty`), verify `tb_website_<name>/.taubyte/build.sh` copies output to `/out`.
- **Static site** (only `index.html` and maybe favicon): use a script that copies files, e.g. `cp index.html /out/` and optionally `[ -e favicon.ico ] && cp favicon.ico /out/`.
- **Node/Vue/React** (has `package.json`): use `npm install`, `npm run build`, then `cp -r dist/* /out` (or yarn equivalent).

See `taubyte-folder-examples.md` (Website section and "Website Build Script (Static)" / "Website Build Script (npm)" in Common Build Script Patterns).

## Pull Website

```bash
tau pull website <name>
```

Examples:

```bash
tau pull website
tau pull website my-site
```

## Checkout Website Branch

```bash
tau checkout website [flags] <name>
```

Flags:

- `--name`, `-n`: Website name (can also be provided as positional argument)
- `--branch`: Branch name to checkout

Examples:

```bash
tau checkout website
tau checkout website --branch feature/new-design my-site
```

## Import Website

```bash
tau import website <name>
```

Examples:

```bash
tau import website
tau import website my-site
```

## Query Website

```bash
tau query website [flags] <name>
```

Flags:

- `--name`, `-n`: Website name (can also be provided as positional argument)
- `--list`: List all websites instead of querying
- `--select`: Force selection prompt

Examples:

```bash
tau query website
tau query website my-site
tau query website --list
```

## List Websites

```bash
tau list websites
```

Aliases:

- `tau list website`

Examples:

```bash
tau list websites
```

## Edit Website

```bash
tau edit website <name>
```

Examples:

```bash
tau edit website
tau edit website my-site
```

## Delete Website

```bash
tau delete website <name>
```

Flags:

- `--yes`, `-y`: Skip confirmation prompts

Examples:

```bash
tau delete website
tau delete website my-site
```

## Non-Interactive Mode

**CRITICAL**: For non-interactive mode, ALWAYS include ALL required flags:

```bash
tau new website --description "[description]" --domains [domain] --paths / --generate-repository --no-private --provider github --branch main --no-embed-token --defaults --yes <name>
```

**Required flags:**

- `--description`: Website description (use empty string `""` if not needed)
- `--domains`: Comma-separated list of domain names
- `--paths`: Comma-separated list of HTTP paths. **CRITICAL (Windows)**: On Windows (Git Bash), prefix the command with `MSYS_NO_PATHCONV=1` to prevent path conversion: `MSYS_NO_PATHCONV=1 tau new website --paths / ...`
- `--generate-repository`: Generate new repository (or omit if using existing)
- `--no-private`: Create public repository (NEVER use `--public`; it is not a valid flag for websites)
- `--provider github`: Git provider (usually github)
- `--branch main`: Branch name (usually main)
- `--no-embed-token`: Disable embedding authentication token
- `--yes`: Skip confirmation prompts

## Common Patterns

### Creating and Deploying Website (Non-Interactive)

```bash
tau new website --description "My website" --domains example.com --paths / --generate-repository --no-private --provider github --branch main --no-embed-token --defaults --yes my-site
tau clone website --no-embed-token --branch main my-site
tau push website --message "Initial website code" my-site
tau query builds
```

### Website with Multiple Domains (Non-Interactive)

```bash
tau new website \
  --description "Multi-domain website" \
  \
  --domains example.com,www.example.com \
  --paths / \
  --generate-repository \
  --no-private \
  --provider github \
  --branch main \
  --no-embed-token \
  --defaults --yes \
  multi-domain
```

### Website with Multiple Paths (Non-Interactive)

```bash
tau new website \
  --description "Multi-path website" \
  \
  --domains example.com \
  --paths /about,/contact \
  --generate-repository \
  --no-private \
  --provider github \
  --branch main \
  --no-embed-token \
  --defaults --yes \
  multi-path
```

### Website from Template (Non-Interactive)

```bash
tau new website \
  --description "My website" \
  \
  --domains example.com \
  --paths / \
  --template template-url \
  --generate-repository \
  --no-private \
  --provider github \
  --branch main \
  --no-embed-token \
  --defaults --yes \
  my-site
```

### Website from Existing Repository (Non-Interactive)

```bash
tau import website --repository-name user/my-site-repo my-site
```

### Branch Management

```bash
tau checkout website --branch feature/new-design my-site
tau push website --message "New design updates" my-site
tau checkout website --branch main my-site
```

## Advanced Usage

### Private Website (Non-Interactive)

```bash
tau new website \
  --description "Private website" \
  \
  --domains example.com \
  --paths / \
  --private \
  --generate-repository \
  --provider github \
  --branch main \
  --no-embed-token \
  --defaults --yes \
  private-site
```

### Website with Custom Repository (Non-Interactive)

```bash
tau new website \
  --description "My website" \
  \
  --domains example.com \
  --paths / \
  --repository-name myorg/my-site-repo \
  --provider github \
  --branch main \
  --no-embed-token \
  --defaults --yes \
  my-site
```

### Website with Template (Non-Interactive)

```bash
tau new website \
  --description "My website" \
  \
  --template https://github.com/example/template \
  --domains example.com \
  --paths / \
  --generate-repository \
  --no-private \
  --provider github \
  --branch main \
  --no-embed-token \
  --defaults --yes \
  my-site
```

### Multiple Websites (Non-Interactive)

```bash
tau new website --description "Main site" --domains example.com --paths / --generate-repository --no-private --provider github --branch main --no-embed-token --defaults --yes main-site
tau new website --description "Blog site" --domains blog.example.com --paths / --generate-repository --no-private --provider github --branch main --no-embed-token --defaults --yes blog-site
tau new website --description "Admin site" --domains admin.example.com --paths / --generate-repository --no-private --provider github --branch main --no-embed-token --defaults --yes admin-site
```

## Troubleshooting

### Error: "Paths [comma separated]: panic: Incorrect function" or CLI prompts for paths even with `--paths /`

**Problem**: The CLI may have issues with path parsing on some systems.

**Solution**: Use normal path syntax with `MSYS_NO_PATHCONV=1` prefix on Windows:

```bash
MSYS_NO_PATHCONV=1 tau new website --description "My website" --domains example.com --paths / --generate-repository --no-private --provider github --branch main --no-embed-token --defaults --yes my-site
```

**Note**:

- `--paths /` → Root path
- `--paths /path1,/path2` → Multiple paths (comma-separated)
- On Windows, always use `MSYS_NO_PATHCONV=1` prefix to prevent path conversion

### Error: "path `C:/Program Files/Git/api/notes` is not valid, should start with /" or path parsing issues (Windows)

**Problem**: On Windows (Git Bash), Git Bash/MSYS automatically converts Unix-style paths that start with `/` to Windows file system paths (e.g., `C:/Program Files/Git/...`), causing path parsing errors.

**Solution**:

- Prefix the command with `MSYS_NO_PATHCONV=1` to prevent path conversion: `MSYS_NO_PATHCONV=1 tau new website --paths / ...`
- For single root path `/`, use `--paths /`
- For multiple paths, use `--paths /path1,/path2` format (comma-separated, no quotes)
- Examples:
  - Root path: `MSYS_NO_PATHCONV=1 tau new website --paths / ...`
  - Multiple paths: `MSYS_NO_PATHCONV=1 tau new website --paths /about,/contact ...`
- **Alternative (persistent)**: Set `export MSYS2_ARG_CONV_EXCL='--paths'` in your `.bashrc` to exclude the `--paths` flag from conversion permanently

### Error: "Website name already exists"

- Use a different name
- Or edit the existing website instead
- Check existing websites: `tau list websites`

### Error: "Domain not found"

```bash
tau new domain --description "Example domain" --fqdn example.com --defaults --yes example.com
tau new website --description "My website" --domains example.com --paths / --generate-repository --no-private --provider github --branch main --no-embed-token --defaults --yes my-site
```

### Error: Website build produces nothing / site not loading after push

- **Cause:** Website `.taubyte/build.sh` may be empty or not copying files to `/out`. The pipeline only serves content from `/out`.
- **Fix:** Edit the website repo's `.taubyte/build.sh`. For static sites: add `cp index.html /out/` (and optional lines for favicon, etc.). For Node/Vue/React: use `npm run build && cp -r dist/* /out`. See "Website repository: .taubyte/build.sh" above and `taubyte-folder-examples.md`.

### Error: "Repository not found"

- Verify repository name is correct
- Check repository visibility (public/private)
- Verify authentication: `tau login`

### Error: "Failed to clone website"

```bash
tau list websites
tau login
tau clone website --name my-site
```

### Error: "Branch not found"

- List available branches
- Use valid branch name
- Create branch if needed

### Error: "opening project config at ... failed" or "Project not cloned locally"

The `tau list websites` command requires the project to be cloned locally first, even though websites have separate repositories.

**Solution:**

```bash
# First clone the project (clones config and code repos)
tau clone project --no-embed-token

# Then you can list websites
tau list websites

# Then clone the website repository separately
tau clone website --no-embed-token my-site
```

**Note**: Projects have config and code repositories, while websites and libraries have their own separate repositories. Each must be cloned separately:

- `tau clone project` - clones project config and code repos
- `tau clone website <name>` - clones website repository
- `tau clone library <name>` - clones library repository
