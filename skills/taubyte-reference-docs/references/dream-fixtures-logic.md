# Dream Fixtures Logic

## Fixtures

Fixtures are pre-configured test scenarios that set up common development environments.

## Running Fixtures

```go
// Run a fixture
err := universe.RunFixture("setBranch", "main")

// Run fixture with multiple parameters
err := universe.RunFixture("pushAll", 
    "project-id-123",
    "main",
)
```

## Available Fixtures

Common fixtures include:
- `setBranch`: Set default branch for services
- `pushAll`: Push all repos to a branch
- `pushConfig`: Push config repository
- `pushCode`: Push code repository
- `pushWebsite`: Push website repository
- `pushLibrary`: Push library repository
- `attachDomain`: Attach default FQDN
- `clearRepos`: Delete all unused repos
- `attachPlugin`: Inject a plugin binary
- `pushSpecific`: Push specific repositories
- `attachProdProject`: Attach a production project
- `importProdProject`: Import and push production project
- `fakeProject`: Push internal project to TNS
- `injectProject`: Inject a project schema
- `compileFor`: Compile code for a resource
- `buildLocalProject`: Build local project

Location: `dream/vars.go` - See `FixtureMap` for complete list

## Registering Custom Fixtures

```go
import "github.com/taubyte/tau/dream"

dream.RegisterFixture("myFixture", func(universe *dream.Universe, params ...interface{}) error {
    // Your fixture logic
    return nil
})
```

## Testing with Fixtures

```go
// Create test universe
universe, err := multiverse.New(dream.UniverseConfig{
    Name:     "test",
    KeepRoot: false, // Temporary for tests
})

// Start services
err = universe.StartAll()
if err != nil {
    return err
}

// Run fixtures
err = universe.RunFixture("setBranch", "test")
err = universe.RunFixture("fakeProject")

// Run tests
// ...

// Cleanup (automatic for temporary universes)
universe.Stop()
```
