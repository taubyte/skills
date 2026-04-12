# Simple Node Logic

## Simple Node Architecture

A simple node is a lightweight peer node that:
- Runs in a single process
- Can host multiple clients simultaneously
- Connects to the universe's P2P network
- Shares the same network infrastructure as full services

## Valid Clients

The following clients can be enabled/disabled on a simple node:

| Client | Purpose | P2P Support |
|--------|---------|--------------|
| `seer` | Service discovery client | Yes |
| `auth` | Authentication client | Yes |
| `patrick` | Project management client | Yes |
| `tns` | Tau Name Service client | Yes |
| `monkey` | Build service client | Yes |
| `hoarder` | Storage client | Yes |
| `substrate` | Runtime execution client | Yes |

Note: These are P2P stream services. HTTP-only services like `gateway` are not available as clients.

## Client Configuration

When creating a simple node:
- Default behavior: All valid clients are enabled
- Enable mode: Only specified clients are enabled
- Disable mode: All clients except specified ones are enabled
- Empty mode: No clients are enabled (can be configured later)

## Port Allocation

Simple nodes are allocated ports automatically:
- Starting port: 50
- Ports increment per simple node
- Each simple node gets a unique P2P port
- Ports are managed by the universe
