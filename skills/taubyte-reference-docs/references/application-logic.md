# Application Logic

## Application Structure

Applications are containers for resources:
- Functions: HTTP, P2P, and PubSub functions
- Databases: Data storage resources
- Domains: Domain configurations
- Libraries: Shared code libraries
- Messaging: Messaging channels
- Services: Service resources
- SmartOps: Smart operations
- Storage: Storage buckets
- Websites: Website resources

## Application Context

- Applications provide a namespace for resources
- Resources belong to either the project (global) or an application (scoped)
- Selected application determines which resources are shown/created by default
- Use `tau current` to see selected application

## Application Selection

- Only one application can be selected at a time
- Selected application is stored in environment/session
- Resources created without explicit application use the selected one
- Global resources (project-level) don't require an application
