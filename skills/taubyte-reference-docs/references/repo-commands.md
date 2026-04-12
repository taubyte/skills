# Repository Commands

## Clone Repository

```bash
tau clone <resource-type> [flags] <name>
```

Resource Types:
- `project`: Project repositories (config and code)
- `library`: Library repositories
- `website`: Website repositories

Flags:
- `--name`, `-n`: Resource name (can also be provided as positional argument)
- `--branch`: Branch to checkout after cloning
- `--embed-token`: Embed authentication token in git config
- `--yes`, `-y`: Skip confirmation prompts

Examples:
```bash
tau clone project
tau clone library my-lib
tau clone website my-site
tau clone library --branch develop my-lib
```

## Push Repository

```bash
tau push <resource-type> [flags] <name>
```

Resource Types:
- `project`: Push project repositories
- `library`: Push library repository
- `website`: Push website repository

Flags:
- `--name`, `-n`: Resource name (can also be provided as positional argument)
- `--commit-message`, `-m`: Commit message (for projects)

Examples:
```bash
tau push project --commit-message "Update configuration"
tau push library my-lib
tau push website my-site
```

## Pull Repository

```bash
tau pull <resource-type> <name>
```

Resource Types:
- `project`: Pull project repositories
- `library`: Pull library repository
- `website`: Pull website repository

Examples:
```bash
tau pull project
tau pull library my-lib
tau pull website my-site
```

## Checkout Repository Branch

```bash
tau checkout <resource-type> [flags] <name>
```

Resource Types:
- `project`: Checkout project branch
- `library`: Checkout library branch
- `website`: Checkout website branch

Flags:
- `--name`, `-n`: Resource name (can also be provided as positional argument)
- `--branch`: Branch name to checkout

Examples:
```bash
tau checkout project --branch develop
tau checkout library --branch feature/new-feature my-lib
tau checkout website --branch feature/new-design my-site
```

## Import Repository

```bash
tau import <resource-type> <name>
```

Resource Types:
- `project`: Import project repositories
- `library`: Import library repository
- `website`: Import website repository

Examples:
```bash
tau import project
tau import library my-lib
tau import website my-site
```
