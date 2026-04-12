# Configuration Logic

## Valid Protocols

| Protocol | Purpose | Effect |
|----------|---------|--------|
| `http` | HTTP protocol | Sets HTTP port |
| `p2p` | Peer-to-peer | Sets P2P port |
| `dns` | DNS protocol | Sets DNS port |
| `https` | HTTPS protocol | Sets both `secure=1` and `http=port` |
| `verbose` | Verbose logging | Enables verbose mode |
| `copies` | Service copies | Sets number of service instances |

## Binding Rules

1. Port Uniqueness: Each port can only be used once per universe
2. Service Validation: Service must be enabled (not disabled)
3. Protocol Validation: Protocol must be in valid list
4. Format Validation: Must match `service@port/protocol` format

## Port Allocation Strategy

### Universe Port Shifts
- Starting shift: 9000
- Ports per universe: 100
- Increment: Each new universe gets +100 port shift
- Calculation: `basePort = 9000 + (universeIndex * 100)`

### Simple Node Ports
- Starting port: 50
- Increment: Each simple node gets next available port
- Allocation: Automatic, sequential

### Service Port Allocation
- Default: System allocates ports automatically
- Custom: Use `--bind` flag to specify ports
- Conflict Detection: System validates for duplicates

### Port Buffer
- Buffer size: 21 ports between protocols
- Purpose: Prevents port conflicts
- Starting port: 100

## Default Values

### Universe Configuration

| Setting | Default Value | Override |
|---------|---------------|----------|
| Universe name | "default" | `--name` flag |
| Host | "127.0.0.1" | `--public` sets to "0.0.0.0" |
| Services | All enabled | `--enable` or `--disable` |
| Simple nodes | "client" | `--simples` flag |
| Fixtures | None | `--fixtures` flag |

### Multiverse Configuration

| Setting | Default Value | Override |
|---------|---------------|----------|
| Multiverse name | "default" | Argument or name |
| API port | 1421 | Not configurable |
| API host | "127.0.0.1" | `--public` sets to "0.0.0.0" |
| Branch | "main" | `--branch` flag |
| Persistent storage | Enabled | `dream.LoadPersistent()` |

### Simple Node Configuration

| Setting | Default Value | Override |
|---------|---------------|----------|
| Simple name | "client" | `--name` flag |
| Clients | All enabled | `--enable` or `--disable` |
| Port | Auto-allocated | System managed |

### Service Configuration

| Setting | Default Value | Override |
|---------|---------------|----------|
| HTTP port | Auto-allocated | `--http` flag or `--bind` |
| P2P port | Auto-allocated | `--bind` with p2p protocol |
| Copies | 1 | `--bind` with copies protocol |

## Configuration Validation

### Service Name Validation
- Must be lowercase
- Must be in valid services list
- Case-sensitive

Valid services: `auth`, `patrick`, `monkey`, `tns`, `hoarder`, `substrate`, `seer`, `gateway`

### Port Validation
- Must be positive integer
- Must be unique per universe
- Must not conflict with system ports

### Protocol Validation
- Must be in valid protocols list
- Case-sensitive

Valid protocols: `http`, `p2p`, `dns`, `https`, `verbose`, `copies`

### Binding Format Validation
- Must match `service@port/protocol` format
- Service must be enabled
- Port must be unique
- Protocol must be valid
