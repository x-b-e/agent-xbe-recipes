---
title: Understanding job schedule shift trucker assignment mechanisms
when: When you need to understand how truckers get assigned to job schedule shifts, or when you're trying to create/update shifts with trucker assignments and encountering limitations
---

## Key Discovery

Job schedule shifts have an `accepted-trucker` field, but this field is **read-only** and cannot be set directly through the CLI when creating or updating shifts.

## What Doesn't Work

```bash
# THESE DO NOT WORK - no --trucker parameter exists
xbe do job-schedule-shifts create --job <job-id> --start-at <start> --end-at <end> --trucker <trucker-id>
xbe do job-schedule-shifts update <shift-id> --trucker <trucker-id>
```

Checking the help for `create` shows no trucker parameter:
```bash
xbe do job-schedule-shifts create --help
# Shows: job, start-at, end-at, is-flexible, etc.
# No trucker or accepted-trucker parameter
```

## How Trucker Assignment Actually Works

Trucker assignment to job schedule shifts happens through a **separate mechanism** not exposed in the CLI:

1. Job schedule shifts are created WITHOUT trucker assignments
2. Truckers get assigned via some other system (tender/acceptance, manual assignment, etc.)
3. The `accepted-trucker` field gets populated by that other system
4. When creating lineup-job-schedule-shifts, you use `--trucker` to copy from the job shift's `accepted-trucker`

## The Lineup Workflow Pattern

```bash
# 1. Job schedule shift exists with accepted-trucker already set (from elsewhere)
xbe view job-schedule-shifts show <shift-id> --json
# Shows: "accepted-trucker-id": "<trucker-id>"

# 2. Create lineup shift, copying the trucker assignment
xbe do lineup-job-schedule-shifts create \
  --lineup <lineup-id> \
  --job-schedule-shift <shift-id> \
  --trucker <trucker-id>  # This copies from job shift's accepted-trucker

# 3. The lineup shift inherits the trucker from job shift
```

## ML Recommendation Tool Integration

The ML recommendation tool (`lineup-job-schedule-shift-trucker-assignment-recommendations`) can help inform trucker assignments:

```bash
# Create a temporary lineup shift to get recommendations
ljss_id=$(xbe do lineup-job-schedule-shifts create \
  --lineup <lineup-id> \
  --job-schedule-shift <shift-id> \
  --json | jq -r '.id')

# Generate ML recommendations
rec_id=$(xbe do lineup-job-schedule-shift-trucker-assignment-recommendations create \
  --lineup-job-schedule-shift $ljss_id \
  --json | jq -r '.id')

# View top candidates
xbe view lineup-job-schedule-shift-trucker-assignment-recommendations show $rec_id --json | \
  jq -r '.candidates[0:5] | .[] | "Rank \(.rank): Trucker \(.trucker_id) - \(.probability*100 | round)%"'
```

## Important Notes

- The `accepted-trucker` field on job-schedule-shifts is **read-only via CLI**
- There is no direct way to assign truckers to job shifts through the CLI commands
- The assignment mechanism exists in a different part of the system (web UI, tender system, etc.)
- The CLI is primarily for **reading** job shift assignments and **copying** them to lineup shifts
