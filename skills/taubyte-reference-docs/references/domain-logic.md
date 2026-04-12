# Domain Logic

## FQDN (Fully Qualified Domain Name)

- Complete domain name including host and domain
- Example: `api.example.com`
- Used for HTTP routing
- Must be unique within the project

## Generated Domains

- Automatically generated FQDN based on project ID
- Format varies by network type:
  - Dream: `{prefix}-{projectId}{count}.{universe}.localtau`
  - Python Test: `{prefix}-{projectId}{count}.tau.link`
  - Remote: `{prefix}-{projectId}{count}.{custom-suffix}`
- Useful for development and testing
- Prefix can be customized

## Domain Validation

- Domains are validated when created (unless generated)
- Validation checks domain registration
- Generated domains skip validation
- Validation response shows registration status

## Certificate Management

- Optional SSL/TLS certificates
- Certificate type selection
- Certificate and key file paths
- Used for HTTPS domains

## Domain Registration

When creating a non-generated domain:
- Domain is validated for registration
- Registration response is displayed
- Domain must be accessible/registerable

## Generated Domain Format

Generated domains follow pattern:
- Project ID (last 8 characters)
- Domain count (incremental)
- Network-specific suffix
- Optional prefix
