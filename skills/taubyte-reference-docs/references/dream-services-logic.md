# Dream Services Logic

## Services

Dream supports all Tau services:
- Auth: Authentication service
- Patrick: Job management service
- TNS: Taubyte Name Service
- Seer: Service discovery
- Monkey: Code compilation service
- Hoarder: Storage service
- Gateway: HTTP gateway service
- Substrate: Runtime execution service

## Accessing Services

```go
// Get first instance of a service
authService := universe.Auth()
patrickService := universe.Patrick()
tnsService := universe.TNS()
seerService := universe.Seer()
monkeyService := universe.Monkey()
hoarderService := universe.Hoarder()
gatewayService := universe.Gateway()
substrateService := universe.Substrate()

// Get all instances of a service
allAuth := universe.AllAuth()
allPatrick := universe.AllPatrick()

// Get service by node ID
pid := "node-id-string"
authService, exists := universe.AuthByPid(pid)
```

## Starting Individual Services

```go
// Start a single service
err := universe.Service("auth", &commonIface.ServiceConfig{
    Others: map[string]int{
        "copies": 1,
    },
})

// Start service with custom ports
err := universe.Service("patrick", &commonIface.ServiceConfig{
    Others: map[string]int{
        "http":   8080,
        "p2p":    8081,
        "copies": 1,
    },
})
```

## Checking Service Availability

```go
// Check if services are provided
err := universe.Provides("auth", "patrick", "tns")
if err != nil {
    // Some services are not available
}
```

## Getting Service Information

```go
// Get service node IDs
pids, err := universe.GetServicePids("auth")

// Get number of service instances
count := universe.ListNumber("auth")
```
