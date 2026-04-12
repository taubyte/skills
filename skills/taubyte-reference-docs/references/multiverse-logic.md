# Multiverse Logic

## Multiverse Architecture

A multiverse:
- Manages multiple universes: Can contain and coordinate multiple universe instances
- Provides persistent storage: Uses `dream.LoadPersistent()` to save state
- Hosts API service: Runs on port 1421 by default
- Integrates MCP: Model Context Protocol service for external tooling
- Manages lifecycle: Handles startup, shutdown, and cleanup of all universes

## Multiverse Name

- Default name: "default" (if not specified)
- Naming: Must be lowercase
- Persistence: Named multiverses can be loaded from persistent storage

## Persistent Storage

When a multiverse is created:
- State is saved to `.cache/dream` directory
- Universes can be restored on subsequent multiverse creation
- Configuration persists across sessions

## Dream API Service

The multiverse runs an HTTP API service:
- Default port: 1421
- Default host: 127.0.0.1 (or 0.0.0.0 with --public)
- URL format: `http://127.0.0.1:1421`
- Purpose: Provides HTTP API for managing universes

## MCP Service

Model Context Protocol service:
- Integrated with the API service
- Provides external tooling integration
- Enables programmatic access to multiverse operations

## Branch Configuration

The `--branch` flag sets the default branch for all services:
- Default: "main"
- Effect: All services resolve resources from this branch
- Override: Can be set per-universe in some cases

## Public vs Private Access

- Private (default): Host set to 127.0.0.1 (localhost only)
- Public (--public): Host set to 0.0.0.0 (all interfaces)
- Use case: Public mode for remote access or containerized deployments

## Debug Mode

When `--debug` is enabled:
- Function call logging is enabled
- Logs are written to temp directory
- Useful for debugging function execution
- Creates `runtime.DebugFunctionCallsLogger`
