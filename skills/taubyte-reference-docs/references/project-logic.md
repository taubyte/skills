# Project Logic

## Project Structure

A Taubyte project consists of:
- Configuration Repository: Contains project configuration, resources, and specifications
- Code Repository: Contains application code, functions, libraries, and websites
- Both repositories follow naming pattern: `tb_<type>_<project-name>`

## Project Visibility

- Public Projects: Repositories are publicly accessible
- Private Projects: Repositories require authentication
- Default is public unless `--private` flag is used

## Repository Operations

Projects support standard git operations:
- Clone: Download project repositories locally
- Push: Upload local changes to remote
- Pull: Download remote changes locally
- Checkout: Switch between branches

## Project Location

- Projects can be cloned to any directory
- Default location is current working directory
- Use `--loc` flag to specify custom location
- Project name is used as subdirectory name

## Dual Repository Handling

The CLI handles both config and code repositories:
- Operations can target both or individually
- Use `--config-only` or `--code-only` for selective operations
- Both repositories must be in sync for full project functionality
