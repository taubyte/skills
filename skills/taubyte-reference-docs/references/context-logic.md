# Context Logic

## Context Components

### Profile
- Authentication profile in use
- Determines authentication credentials
- Set via `tau login`
- Stored in session

### Project
- Currently selected project
- Resources are created in this project
- Set via `tau select project`
- Stored in session

### Application
- Currently selected application (optional)
- Resources can be scoped to application
- Set via `tau select application`
- Stored in session
- Can be "(none)" if no application selected

### Network
- Network Type: Type of network (dream, remote)
- Network: Network URL or universe name
- Set via `tau select network`
- Stored in profile

## Context Display

- Shows "(none)" for unset values
- Table format for easy reading
- Updates automatically when context changes
- Reflects current session state
