# Fixture Commands

## Inject Fixture

```bash
dream inject fixture <fixture-name> [flags]
```

Common Flags:
- `--universe`, `-u` (string): Target universe name (default: "default")
- Fixture-specific flags vary by fixture

Examples:
```bash
dream inject fixture set-branch --name main --universe test
dream inject fixture push-all --project-id test-project --branch main
dream inject fixture attach-plugin --paths /path/to/plugin1.wasm,/path/to/plugin2.wasm
```

## Inject Fixtures During Universe Creation

```bash
dream new universe [name] --fixtures <fixture1>,<fixture2>,...
```

Examples:
```bash
dream new universe test --fixtures set-branch,push-all
dream new universe dev --fixtures set-branch --enable auth,patrick
```

## Available Fixtures

### setBranch

```bash
dream inject fixture set-branch
```

Flags:
- `--name`, `-n` (string, required): Branch name to set

Examples:
```bash
dream inject fixture set-branch --name main
dream inject fixture set-branch --name develop --universe test
```

### pushAll

```bash
dream inject fixture push-all
```

Flags:
- `--project-id`, `--pid` (string, optional): Project ID (defaults to test project)
- `--branch`, `-b` (string, optional): Branch name (defaults to "main")

Examples:
```bash
dream inject fixture push-all
dream inject fixture push-all --project-id my-project --branch develop
```

### attachPlugin

```bash
dream inject fixture attach-plugin
```

Flags:
- `--paths`, `-p` (string, required): Comma-separated list of binary paths

Examples:
```bash
dream inject fixture attach-plugin --paths /path/to/plugin.wasm
dream inject fixture attach-plugin --paths /path/to/plugin1.wasm,/path/to/plugin2.wasm
```

### pushSpecific

```bash
dream inject fixture push-specific
```

Flags:
- `--repository-id`, `--rid` (string, required): Repository ID
- `--repository-fullname`, `--fn` (string, required): Repository full name (e.g., "taubyte-test/tb_repo")
- `--project-id`, `--pid` (string, optional): Project ID (defaults to test project)
- `--branch`, `-b` (string, optional): Branch name (defaults to "main")

Examples:
```bash
dream inject fixture push-specific --rid repo-id --fn taubyte-test/tb_repo
dream inject fixture push-specific --rid repo-id --fn taubyte-test/tb_repo --branch develop
```
