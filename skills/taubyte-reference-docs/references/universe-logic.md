# Universe Logic

## Universe States

A universe can be in one of four states:
- Stopped: Universe is not running
- Starting: Universe is in the process of starting services
- Running: Universe is fully operational with all services running
- Stopping: Universe is shutting down

## Valid Services

The following services can be enabled/disabled in a universe:
- `auth`: Authentication service
- `patrick`: Project management service
- `monkey`: Build and compilation service
- `tns`: Tau Name Service
- `hoarder`: Storage service
- `substrate`: Runtime execution service
- `seer`: Service discovery
- `gateway`: HTTP gateway service

## Service Binding Syntax

Service bindings use the format: `service@port/protocol`

Protocols:
- `http`: HTTP protocol
- `p2p`: Peer-to-peer protocol
- `dns`: DNS protocol
- `https`: HTTPS protocol (sets both secure=1 and http=port)
- `verbose`: Verbose logging
- `copies`: Number of service copies

Examples:
- `patrick@8080/http`: Patrick service on HTTP port 8080
- `substrate@9000/p2p`: Substrate service on P2P port 9000
- `gateway@443/https`: Gateway service on HTTPS port 443
- `seer@5000/verbose`: Seer service with verbose logging on port 5000

## Simple Nodes

Simple nodes are lightweight nodes that run multiple clients in a single process. By default, a simple node named "client" is created with all clients enabled.
