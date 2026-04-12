# Builds & Logs Logic

## Build Jobs

- Represent compilation and deployment jobs
- Each job has a unique ID
- Jobs have timestamps
- Jobs can be filtered by time range
- Default time range: 7 days

## Job Logs

- Detailed output from build jobs
- Multiple log files per job (one per resource)
- Can be viewed in terminal or saved to files
- Log files named by resource ID

## Time Range Filtering

- Filter builds by time range
- Format: duration string
- Examples:
  - `1h` - last hour
  - `24h` - last 24 hours
  - `7d` - last 7 days
  - `2w` - last 2 weeks
- Default: 7 days
