# Testing Commands

## Development Environment Setup

### Basic Development Setup
```bash
# Terminal 1: Create multiverse
dream new multiverse

# Terminal 2: Create development universe
dream new universe dev --enable auth,patrick,substrate
```

### Isolated Development Universe
```bash
dream new universe dev \
  --enable auth,patrick,substrate \
  --fixtures set-branch \
  --simples dev-client
```

### Full-Stack Development Environment
```bash
dream new universe full-dev \
  --enable auth,patrick,monkey,tns,hoarder,substrate,seer,gateway \
  --fixtures set-branch,push-all \
  --simples dev-client,test-client
```

## Integration Testing Patterns

### Test Universe Creation
```bash
dream new universe test-$(date +%s) --empty
dream new universe test-$(date +%s) \
  --enable auth,patrick,substrate \
  --fixtures set-branch
```

### Parallel Test Execution
```bash
# Terminal 1: Create multiverse
dream new multiverse

# Terminal 2: Create multiple test universes
dream new universe test1
dream new universe test2
dream new universe test3
dream new universe test4
```

### Test Isolation
```bash
dream new universe test-${TEST_ID} --empty
dream kill universe test-${TEST_ID}
```

## Fixture-Based Testing

### Setting Up Test Data
```bash
dream inject fixture set-branch --name test-branch --universe test
dream inject fixture push-all \
  --project-id test-project \
  --branch test-branch \
  --universe test
```

### Custom Test Fixtures
```bash
dream new universe integration-test
dream inject fixture set-branch --name test --universe integration-test
dream inject fixture push-all \
  --project-id test-project \
  --branch test \
  --universe integration-test
dream inject fixture attach-plugin \
  --paths ./test-plugins/test.wasm \
  --universe integration-test
```

### Repository Testing
```bash
dream inject fixture push-specific \
  --repository-id test-repo \
  --repository-fullname org/test-repo \
  --project-id test-project \
  --branch test-branch \
  --universe test
```

## Test Data Management

### Fresh Test Data Per Run
```bash
dream new universe test-run-$(uuidgen) --empty
dream inject fixture set-branch --name test --universe test-run-*
dream kill universe test-run-*
```

### Persistent Test Data
```bash
dream new universe persistent-test \
  --enable auth,patrick,substrate \
  --fixtures set-branch,push-all
```

### Test Data Cleanup
```bash
dream kill universe test-universe
dream kill universe test-*
```
