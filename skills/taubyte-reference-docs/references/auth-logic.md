# Authentication Logic

## Profile Configuration

Each profile stores:
- Provider: Git provider (e.g., "github")
- Token: Authentication token for the provider
- Default: Whether this profile is the default
- GitUsername: Extracted from token
- GitEmail: Extracted from token
- NetworkType: Network type associated with the profile
- Network: Network URL associated with the profile

## Authentication Profiles

Profiles allow you to manage multiple authentication credentials:
- Each profile is associated with a git provider (currently GitHub)
- Profiles can be named for easy identification
- One profile can be set as the default
- Profiles are stored in the tau configuration

## Default Profile

- The default profile is automatically selected when no profile is specified
- Only one profile can be default at a time
- Setting a new profile as default removes the default flag from the previous default
- If no profiles exist, the first created profile becomes the default

## Token Management

- Tokens are stored securely in the tau configuration
- Tokens are used to authenticate with Taubyte clouds
- Git username and email are automatically extracted from tokens when possible
- Tokens must be valid for the specified provider

## Token Extraction

The CLI automatically extracts git information from tokens:
- Git username is extracted when possible
- Git email is extracted when possible
- If extraction fails, a warning is shown but profile creation continues

## Provider Support

Currently supported providers:
- GitHub: Primary and only supported provider
