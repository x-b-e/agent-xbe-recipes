---
title: Tracing lineup dispatch timeline and dispatcher activity
when: When you need to understand the timeline of when lineup shifts were dispatched, who performed the dispatches, or when you need to trace the sequence of dispatch events throughout a day including timezone conversions
---

## Overview

This recipe shows how to trace the complete timeline of dispatches for a lineup, including who dispatched shifts and when (in local time).

## Key Concepts

- **Lineups** contain job schedule shifts that need to be dispatched
- **Lineup-job-schedule-shifts** are the individual shifts within a lineup
- **Lineup-dispatches** are the events where dispatchers send offers to trucking companies
- **Lineup-dispatch-lineup-job-schedule-shifts** links dispatches to the specific shifts being dispatched
- All timestamps are stored in UTC and need timezone conversion for local interpretation

## Steps

### 1. Find the lineup

```bash
# List lineups for a broker
xbe view lineups list --broker <broker-id> --json --limit 100

# Get details for a specific lineup including time window
xbe view lineups show <lineup-id> --json | jq '{id, "start-at-min", "start-at-max"}'
```

### 2. Get all shifts in the lineup

```bash
# List all lineup-job-schedule-shifts for the lineup
xbe view lineup-job-schedule-shifts list --lineup <lineup-id> --json --limit 1000
```

### 3. Find all dispatch events for the lineup

```bash
# Find dispatches by getting dispatch IDs from the junction table
xbe view lineup-dispatch-lineup-job-schedule-shifts list \
  --lineup-job-schedule-shift <ljss-id> --json --limit 100 \
  | jq -r '.[]."lineup-dispatch-id"' | sort -u

# Or to find all dispatches across all shifts in the lineup:
for ljss_id in $(xbe view lineup-job-schedule-shifts list --lineup <lineup-id> --json --limit 1000 | jq -r '.[].id'); do
  xbe view lineup-dispatch-lineup-job-schedule-shifts list \
    --lineup-job-schedule-shift $ljss_id --json 2>/dev/null \
    | jq -r '.[]."lineup-dispatch-id"'
done | sort -u
```

### 4. Get dispatch details including timestamps and dispatchers

```bash
# Get details for each dispatch
xbe view lineup-dispatches show <dispatch-id> --json \
  | jq '{id, "created-at", "created-by-id", "auto-offer-customer-tender", "auto-offer-trucker-tender"}'

# Get dispatcher name
xbe view users show <user-id> --json | jq '{name}'
```

### 5. Count shifts per dispatch

```bash
# Count how many shifts were in each dispatch
xbe view lineup-dispatch-lineup-job-schedule-shifts list \
  --lineup-dispatch <dispatch-id> --json | jq 'length'
```

### 6. Convert UTC timestamps to local time

```bash
# First, determine the timezone from a job schedule shift
xbe view lineup-job-schedule-shifts show <ljss-id> --json | jq '."job-schedule-shift-id"'
xbe view job-schedule-shifts show <jss-id> --json | jq '."time-zone-id"'

# Convert UTC timestamp to local time (example for America/Chicago)
# The lineup start-at times include 'Z' suffix indicating UTC
# For Central Daylight Time (CDT), subtract 5 hours from UTC
# For Central Standard Time (CST), subtract 6 hours from UTC
```

## Complete Example Workflow

```bash
# 1. Find lineup for a specific date
xbe view lineups list --broker <broker-id> --json --limit 100 \
  | jq '.[] | select(."start-at-min" | startswith("<date>"))'

# 2. Get all shifts in the lineup
xbe view lineup-job-schedule-shifts list --lineup <lineup-id> --json --limit 1000 > shifts.json

# 3. Find all unique dispatch IDs
cat shifts.json | jq -r '.[].id' | while read ljss_id; do
  xbe view lineup-dispatch-lineup-job-schedule-shifts list \
    --lineup-job-schedule-shift $ljss_id --json 2>/dev/null \
    | jq -r '.[]."lineup-dispatch-id"'
done | sort -u > dispatch_ids.txt

# 4. Get dispatch details with shift counts
cat dispatch_ids.txt | while read dispatch_id; do
  dispatch_info=$(xbe view lineup-dispatches show $dispatch_id --json)
  created_at=$(echo "$dispatch_info" | jq -r '."created-at"')
  created_by_id=$(echo "$dispatch_info" | jq -r '."created-by-id"')
  shift_count=$(xbe view lineup-dispatch-lineup-job-schedule-shifts list \
    --lineup-dispatch $dispatch_id --json | jq 'length')
  
  # Get dispatcher name
  dispatcher_name=$(xbe view users show $created_by_id --json | jq -r '.name')
  
  echo "Dispatch $dispatch_id: $created_at, $shift_count shifts, by $dispatcher_name"
done | sort -k3

# 5. Determine timezone for conversion
first_ljss_id=$(cat shifts.json | jq -r '.[0].id')
jss_id=$(xbe view lineup-job-schedule-shifts show $first_ljss_id --json | jq -r '."job-schedule-shift-id"')
timezone=$(xbe view job-schedule-shifts show $jss_id --json | jq -r '."time-zone-id"')
echo "Timezone: $timezone"
```

## Key Insights

1. **Multiple dispatch events**: A single lineup typically has many dispatch events throughout the day as shifts are added, adjusted, or re-dispatched

2. **Bulk + incremental pattern**: Often there's one large dispatch (e.g., 50+ shifts) followed by smaller dispatches (1-10 shifts) for adjustments

3. **Timing patterns**: 
   - Bulk dispatches often happen the afternoon/evening before the work day
   - Incremental dispatches happen throughout the work day as needs change

4. **Multiple dispatchers**: Different dispatchers may handle initial bulk dispatch vs day-of adjustments

5. **UTC vs local time**: All timestamps are UTC - always check timezone and convert for meaningful interpretation of "when" dispatches happened relative to the work day

## Limitations

- No direct way to query all dispatches for a lineup - must iterate through lineup-job-schedule-shifts
- Created-at timestamps on job-schedule-shifts are not exposed, so can't determine when shifts were originally planned
- Timezone information is on job-schedule-shifts, not on lineups or dispatches
