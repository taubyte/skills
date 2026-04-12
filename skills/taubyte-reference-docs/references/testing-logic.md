# Testing Logic

## Testing Patterns

- Test isolation: Each test gets its own universe
- Parallel execution: Multiple universes run independently
- Fixture-based setup: Use fixtures for common test scenarios
- Fresh data: Create new universes for each test run
- Persistent data: Reuse universes across test runs

## CI/CD Integration

- GitHub Actions: Use Dream CLI in CI pipelines
- Test universes: Create isolated test environments
- Cleanup: Kill universes after tests complete
