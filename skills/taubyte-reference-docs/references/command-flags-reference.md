# Command Flags Reference

## Project Commands

### tau new project [flags] <name>
**Flags Available:**
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

### tau clone project
**Flags Available:**
- `--location`, `--loc`: Directory location
- `--branch`, `-b`: Branch to checkout
- `--embed-token`, `-e`: Embed authentication token (use `--no-embed-token` or `--no-e` to disable)
- `--select`: Force selection prompt
- `--yes`, `-y`: Skip confirmation prompts only (does not skip boolean prompts like embed-token)

**Interactive Prompts:**
- Embed Git Token: True/False (can be bypassed with --embed-token or --no-embed-token)
- Select Branch: (can be bypassed with --branch)

### tau push project
**Flags Available:**
- `--message`, `-m`: Commit message
- `--config-only`, `--config`: Push only config repo
- `--code-only`, `--code`: Push only code repo

**Interactive Prompts:**
- Commit Message: (can be bypassed with --message)

## Domain Commands

### tau new domain [flags] <name>
**Flags Available:**
- `--description`, `-d`: Domain description
- `--generated-fqdn`, `--g-fqdn`: Generate an FQDN based on the project ID
- `--generated-fqdn-prefix`, `--g-prefix`: Prefix to use when generating an FQDN
- `--fqdn`, `-f`: Fully Qualified Domain Name
- `--cert-type`, `--type`: Certificate type
- `--certificate`, `-c`: Certificate file path
- `--key`, `-k`: Private key file path
- `--yes`, `-y`: Skip confirmation prompts

**Interactive Prompts:**
- Description: (can be bypassed with --description)
- Generate FQDN: True/False (can be bypassed with --generated-fqdn)
- FQDN prefix: (can be bypassed with --generated-fqdn-prefix)
- Certificate type: (can be bypassed with --cert-type)
- Confirm creation: Yes/No (can be bypassed with --yes)

### tau edit domain [flags] <name>
**Flags Available:**
- `--description`, `-d`: Domain description
- `--fqdn`, `-f`: FQDN
- `--cert-type`, `--type`: Certificate type
- `--certificate`, `-c`: Certificate file path
- `--key`, `-k`: Private key file path
- `--yes`, `-y`: Skip confirmation prompts

**Interactive Prompts:**
- Description: (can be bypassed with --description)
- FQDN: (can be bypassed with --fqdn)
- Certificate type: (can be bypassed with --cert-type)
- Confirm edit: Yes/No (can be bypassed with --yes)

## Application Commands

### tau new application [flags] <name>
**Flags Available:**
- `--description`, `-d`: Application description
- `--yes`, `-y`: Skip confirmation prompts

**Interactive Prompts:**
- Description: (can be bypassed with --description)
- Confirm creation: Yes/No (can be bypassed with --yes)

## Function Commands

### tau new function [flags] <name>
**Flags Available:**
- `--description`, `-d`: Function description
- `--timeout`, `--ttl`: Time to live for an instance
- `--memory`, `--me`: Max memory either in form 10GB or 10
- `--memory-unit`, `--mu`: Unit if not provided with memory
- `--type`: Function type (http, p2p, pubsub)
- `--source`: Path within the code folder or a library repository
- `--call`, `--ca`: Exported function to call
- `--template`: Template URL or name
- `--language`, `--lang`: Template language
- `--use-template`: Use template
- `--yes`, `-y`: Skip confirmation prompts

**HTTP Function Flags:**
- `--method`, `-m`: HTTP method (GET, POST, PUT, DELETE, etc.)
- `--domains`: List of domains (comma separated) by name or FQDN
- `--paths`, `-p`: HTTP paths to use for the endpoint
- `--generate-domain`, `-g`: Generates a new domain if no domains found

**P2P Function Flags:**
- `--protocol`, `--pr`: Protocol to use for the endpoint
- `--command`, `--cmd`: Command to execute
- `--local`, `-l`: Enable local execution

**PubSub Function Flags:**
- `--channel`, `--ch`: Channel to subscribe to
- `--local`, `-l`: Enable local execution

**Interactive Prompts:**
- Use Template: True/False (can be bypassed with --use-template)
- Description: (can be bypassed with --description)
- Time To Live: (can be bypassed with --timeout)
- Memory: (can be bypassed with --memory)
- Type: (can be bypassed with --type)
- Domains: (can be bypassed with --domains)
- Method: (can be bypassed with --method)
- Paths: (can be bypassed with --paths)
- Code source: (can be bypassed with --source)
- Call: (can be bypassed with --call)
- Confirm creation: Yes/No (can be bypassed with --yes)

## Database Commands

### tau new database [flags] <name>
**Flags Available:**
- `--description`, `-d`: Database description
- `--regex`, `-r`: Match using regex
- `--match`, `-m`: Match pattern ([^regex] if configured or /path/to/match)
- `--local`, `-l`: Enable local execution
- `--encryption`: Enable encryption
- `--encryption-key`, `-k`, `--key`: Encryption key
- `--min`: Minimum replicas to keep
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

## Storage Commands

### tau new storage [flags] <name>
**Flags Available:**
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

## Messaging Commands

### tau new messaging [flags] <name>
**Flags Available:**
- `--description`, `-d`: Resource description
- `--local`, `-l`: Enable local execution
- `--regex`, `-r`: Match using regex
- `--match`, `-m`: Match pattern ([^regex] if configured or /path/to/match)
- `--mqtt`: Enable MQTT protocol
- `--web-socket`, `--ws`: Enable joining the pubsub through websocket
- `--yes`, `-y`: Skip confirmation prompts

**Interactive Prompts:**
- Description: (can be bypassed with --description)
- Local: (can be bypassed with --local)
- Use Regex For Match: True/False (can be bypassed with --regex)
- Channel Match: (can be bypassed with --match)
- Enable MQTT bridge: (can be bypassed with --mqtt)
- Enable WebSocket bridge: (can be bypassed with --web-socket)
- Confirm creation: Yes/No (can be bypassed with --yes)

## Website Commands

### tau new website [flags] <name>
**Flags Available:**
- `--description`, `-d`: Website description
- `--template`: Template URL or name
- `--domains`: List of domains (comma separated) by name or FQDN
- `--paths`, `-p`: HTTP paths to use for the endpoint
- `--provider`: Git provider
- `--repository-name`, `--repo-n`: Repository name
- `--repository-id`, `--repo-id`: Repository ID
- `--branch`, `-b`: Branch name
- `--clone`: Clone repository after creation
- `--embed-token`, `-e`: Embed authentication token (use `--no-embed-token` or `--no-e` to disable)
- `--generate-repository`, `-g`: Generate new repository
- `--private`: Create private repository; use `--no-private` for public repos. **NEVER use `--public`** (not a valid website flag).
- `--yes`, `-y`: Skip confirmation prompts only (does not skip boolean prompts like embed-token)

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

### tau clone website [flags] <name>
**Flags Available:**
- `--branch`, `-b`: Branch to checkout
- `--embed-token`, `-e`: Embed authentication token (use `--no-embed-token` or `--no-e` to disable)

**Interactive Prompts:**
- Embed Git Token: True/False (can be bypassed with --embed-token or --no-embed-token)
- Select Branch: (can be bypassed with --branch)

### tau push website <name>
**Flags Available:**
- `--message`, `-m`: Commit message

**Interactive Prompts:**
- Commit Message: (can be bypassed with --message)

## Library Commands

### tau new library [flags] <name>
**Flags Available:**
- `--description`, `-d`: Library description
- `--template`: Template URL or name
- `--provider`: Git provider
- `--path`, `-p`: Repository path
- `--repository-name`, `--repo-n`: Repository name
- `--repository-id`, `--repo-id`: Repository ID
- `--clone`: Clone repository after creation
- `--embed-token`, `-e`: Embed authentication token (use `--no-embed-token` or `--no-e` to disable)
- `--generate-repository`, `-g`: Generate new repository
- `--private`: Create private repository
- `--branch`, `-b`: Branch name
- `--yes`, `-y`: Skip confirmation prompts only (does not skip boolean prompts like embed-token)

**Interactive Prompts:**
- Description: (can be bypassed with --description)
- Git Provider: (can be bypassed with --provider)
- Generate Repository: True/False (can be bypassed with --generate-repository)
- Private: True/False (can be bypassed with --private)
- Select Template: (can be bypassed with --template)
- Path: (can be bypassed with --path)
- Branch: (can be bypassed with --branch)
- Confirm creation: Yes/No (can be bypassed with --yes)
- Embed Git Token: True/False (can be bypassed with --embed-token or --no-embed-token)

### tau clone library [flags] <name>
**Flags Available:**
- `--branch`, `-b`: Branch to checkout
- `--embed-token`, `-e`: Embed authentication token (use `--no-embed-token` or `--no-e` to disable)

**Interactive Prompts:**
- Embed Git Token: True/False (can be bypassed with --embed-token or --no-embed-token)
- Select Branch: (can be bypassed with --branch)

### tau push library <name>
**Flags Available:**
- `--message`, `-m`: Commit message

**Interactive Prompts:**
- Commit Message: (can be bypassed with --message)

## Authentication Commands

### tau login --new [profile-name]
**Flags Available:**
- `--new`: Force creation of a new profile
- `--set-default`, `-d`: Set the newly created profile as the default
- `--token`, `-t`: Provide authentication token directly
- `--provider`, `-p`: Specify the git provider (default: "github")
- `--name`: Specify the profile name directly (or provide as argument)

**Interactive Prompts:**
- Use as default: True/False (can be bypassed with --set-default)
- Git Provider: (can be bypassed with --provider)
- Get token with: Select authentication method (cannot bypass if using web auth)

## Context Commands

### tau select cloud
**Flags Available:**
- `--fqdn`, `-f`: FQDN of remote network to connect to
- `--universe`, `-u`: Dream universe to connect to

## Build Commands

### tau query builds
**Flags Available:**
- `--since`, `-t`, `-s`: Time range filter (e.g., "24h", "7d")

**Interactive Prompts:**
- None (all flags are optional)

### tau query logs
**Flags Available:**
- `--jid`: Job ID (required)
- `--output`, `-o`: Output directory for log files
