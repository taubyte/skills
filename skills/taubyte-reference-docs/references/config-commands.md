# Configuration Commands

## Service Binding Format

Syntax: `service@port/protocol`

Components:
- service: Service name (auth, patrick, monkey, tns, hoarder, substrate, seer, gateway)
- port: Port number (integer)
- protocol: Protocol type (http, p2p, dns, https, verbose, copies)

Examples:
```bash
dream new universe test --bind patrick@8080/http
dream new universe test --bind substrate@9000/p2p
dream new universe test --bind gateway@443/https
dream new universe test --bind seer@5000/verbose
dream new universe test --bind node@1/copies
```

## Multiple Bindings

```bash
dream new universe test \
  --bind patrick@8080/http \
  --bind substrate@9000/p2p \
  --bind gateway@443/https \
  --bind seer@5000/verbose
```

## Binding Without Protocol

```bash
dream new universe test --bind patrick@8080
```

## Configuration Patterns

### Minimal Configuration
```bash
dream new universe minimal --empty
```

### Default Configuration
```bash
dream new universe default
```

### Custom Port Configuration
```bash
dream new universe custom \
  --bind patrick@8080/http \
  --bind substrate@9000/p2p \
  --bind gateway@443/https
```

### Service Selection Configuration
```bash
dream new universe selected --enable auth,patrick,substrate
dream new universe excluded --disable monkey,hoarder
```

### Multi-Simple Configuration
```bash
dream new universe multi \
  --simples client1,client2,client3
```

### Fixture Configuration
```bash
dream new universe with-fixtures \
  --fixtures set-branch,push-all
```

### Combined Configuration
```bash
dream new universe complex \
  --enable auth,patrick,substrate,seer \
  --bind patrick@8080/http \
  --bind substrate@9000/p2p \
  --fixtures set-branch \
  --simples test-client,dev-client
```

## Advanced Configuration

### Empty Universe Setup
```bash
dream new universe empty --empty
dream inject service auth --universe empty
dream inject service patrick --universe empty
dream inject simple client --universe empty
```

### Port Conflict Avoidance
```bash
dream new universe test1 \
  --bind patrick@8080/http \
  --bind substrate@9000/p2p
dream new universe test2 \
  --bind patrick@8081/http \
  --bind substrate@9001/p2p
```

### Service Binding Strategies

#### HTTP-Only Services
```bash
dream new universe http-only \
  --enable patrick,substrate,seer,auth,gateway \
  --bind patrick@8080/http \
  --bind gateway@443/https
```

#### P2P-Only Services
```bash
dream new universe p2p-only \
  --enable monkey,tns,hoarder \
  --bind monkey@5000/p2p \
  --bind tns@5001/p2p
```

#### Mixed Protocol Services
```bash
dream new universe mixed \
  --bind patrick@8080/http \
  --bind substrate@9000/p2p \
  --bind seer@5000/verbose
```

### Branch Configuration
Branch configuration is handled via universe creation and fixtures. The multiverse must be created first.

### Public Access Configuration
Public access is configured via universe creation with service bindings. The multiverse must be created first.

## Configuration Examples

### Development Setup
```bash
# Terminal 1: Create multiverse
dream new multiverse

# Terminal 2: Create development universe
dream new universe dev \
  --enable auth,patrick,substrate \
  --bind patrick@8080/http \
  --fixtures set-branch
```

### Testing Setup
```bash
# Terminal 1: Create multiverse
dream new multiverse

# Terminal 2: Create test universe
dream new universe test \
  --enable auth,patrick,substrate \
  --bind patrick@8080/http \
  --fixtures set-branch,push-all
```

### Production-Like Setup
```bash
# Terminal 1: Create multiverse
dream new multiverse

# Terminal 2: Create production-like universe
dream new universe prod \
  --enable auth,patrick,substrate,gateway \
  --bind gateway@443/https
```

### Minimal Testing Setup
```bash
dream new universe minimal \
  --enable auth,patrick \
  --empty
```
