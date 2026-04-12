# Builds & Logs Commands

## Query Builds

```bash
tau query builds [flags]
```

Aliases:

- `tau list builds`
- `tau list jobs`

Flags:

- `--since`, `-t`, `-s`: Time range filter (e.g., "24h", "7d")

**Interactive Prompts:**

- None (all flags are optional)

Examples:

```bash
tau query builds
tau query builds --since 24h
tau query builds --since 7d
tau query builds --since 2w
tau list builds
tau list jobs
```

## Query Logs

```bash
tau query logs [resource-or-job-id]
tau query logs [flags]
```

You can pass the job or resource ID as a **positional argument** (e.g. from a failure message or `tau query builds`), or use `--jid`.

Flags:

- `--jid`: Job ID (alternative to positional ID)
- `--output`, `-o`: Output directory for log files

Examples:

```bash
# Positional ID (e.g. resource CID from error or builds list)
tau query logs QmU4P1Mw3Bqpgq7AXDZpXyb3rsvoJKi1C4iWhso1w9sZet
tau query logs <resource-or-job-id>

# With --jid
tau query logs --jid <job-id>
tau query logs --jid <job-id> --output ./logs
tau query logs --jid abc123 --output /tmp/build-logs
```

## Common Patterns

### Viewing Recent Builds

```bash
tau query builds
tau query builds --since 24h
```

### Getting Build Logs

```bash
tau query builds
tau query logs --jid <job-id>
```

### Saving Logs to Files

```bash
tau query logs --jid <job-id> --output ./logs
```

### Build History Analysis

```bash
tau query builds --since 1h
tau query builds --since 24h
tau query builds --since 7d
tau query builds --since 30d
```

## Advanced Usage

### Multiple Time Ranges

```bash
tau query builds --since 1h
tau query builds --since 24h
tau query builds --since 7d
```

### Log Analysis Workflow

```bash
tau query builds --since 24h
tau query logs --jid <job-id>
tau query logs --jid <job-id> --output ./analysis-logs
```

### Automated Log Collection

```bash
for jid in $(tau query builds --since 24h | grep -o '[a-f0-9]\{24\}'); do
  tau query logs --jid $jid --output ./logs/$jid
done
```

## Troubleshooting

### Error: "Job id not set"

- Pass the job or resource ID either as a positional argument or with `--jid`. Get the ID from the failure message or from `tau query builds`.

```bash
tau query logs <resource-or-job-id>
# or
tau query logs --jid <job-id>
tau query builds
```

- Full workflow: `check-logs-debug.md`.

### Error: "Job not found"

- Verify job ID is correct
- Check job exists in time range: `tau query builds --since 30d`
- Job may have expired or been deleted

### No Builds Showing

```bash
tau query builds --since 30d
tau current
tau query builds --since 365d
```

### Logs Not Available

- Try querying more recent jobs
- Check job status in builds query
- Verify job completed successfully

### Output Directory Issues

```bash
tau query logs --jid <job-id> --output /tmp/logs
```
