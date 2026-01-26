---
title: Tracing the complete lifecycle of a lineup dispatch
when: When you need to understand the full sequence of events from job planning through dispatch fulfillment for a lineup, or when you need to trace who did what and when in the lineup/dispatch process, or when you need to understand dispatch timing patterns
---

# Tracing the complete lifecycle of a lineup dispatch

## Overview

Lineups represent a collection of scheduled truck shifts for a specific time window. Understanding the full lifecycle from job planning through dispatch requires working backward through multiple related resources and understanding timing patterns.

## Key Resources and Relationships

```
job-production-plans (jobs)
  ↓ generates
job-schedule-shifts (individual truck shifts)
  ↓ added to
lineup-job-schedule-shifts (shifts in a lineup)
  ↓ dispatched via
lineup-dispatches (dispatch events)
  ↓ creates
lineup-dispatch-shifts (individual shifts in a dispatch)
```

## Step-by-Step Process

### 1. Find the lineup

```bash
# Find lineups for a broker
xbe view lineups list --broker <broker-id> --json --limit 100

# Get lineup details including time window
xbe view lineups show <lineup-id> --json
```

### 2. Examine shifts in the lineup

```bash
# Get all shifts in the lineup
xbe view lineup-job-schedule-shifts list --lineup <lineup-id> --json --limit 1000

# Count total shifts
xbe view lineup-job-schedule-shifts list --lineup <lineup-id> --json --limit 1000 | jq 'length'

# See which truckers are assigned
xbe view lineup-job-schedule-shifts list --lineup <lineup-id> --fields job-schedule-shift --limit 20
```

### 3. Find all dispatches for the lineup

**Important limitation**: The `lineup-dispatches` resource does NOT support filtering by `--lineup` despite the relationship. You must use `lineup-dispatch-shifts` to work backward.

```bash
# Find dispatch-shifts for this lineup's shifts
# First get lineup shift IDs
lineup_shift_ids=$(xbe view lineup-job-schedule-shifts list --lineup <lineup-id> --json --limit 1000 | jq -r '.[].id')

# Then find which dispatches included these shifts
for shift_id in $lineup_shift_ids; do
  xbe view lineup-dispatch-shifts list --lineup-job-schedule-shift $shift_id --json 2>/dev/null
done | jq -s 'map(."lineup-dispatch-id") | unique'
```

### 4. Analyze dispatch timing patterns

**Critical discovery**: Use `created-at` filtering on `lineup-dispatch-shifts` to understand when dispatches occurred, since `lineup-dispatches` may not expose these filters.

```bash
# Find which dispatches happened on a specific date
for dispatch_id in <dispatch-id-1> <dispatch-id-2> <dispatch-id-3>; do
  count=$(xbe view lineup-dispatch-shifts list \
    --created-at-min "<date>T00:00:00Z" \
    --created-at-max "<date>T23:59:59Z" \
    --lineup-dispatch $dispatch_id \
    --json 2>/dev/null | jq 'length')
  if [ "$count" != "0" ]; then
    echo "Dispatch $dispatch_id: $count shifts on <date>"
  fi
done

# Get hourly dispatch timeline
for hour in {00..23}; do
  for dispatch_id in <dispatch-ids>; do
    count=$(xbe view lineup-dispatch-shifts list \
      --created-at-min "<date>T${hour}:00:00Z" \
      --created-at-max "<date>T${hour}:59:59Z" \
      --lineup-dispatch $dispatch_id \
      --json 2>/dev/null | jq 'length' 2>/dev/null)
    if [ "$count" != "0" ] && [ -n "$count" ]; then
      who=$(xbe view lineup-dispatches show $dispatch_id --fields created-by 2>/dev/null | tail -1 | awk '{print $2, $3}')
      echo "<date>, ${hour}:xx | Dispatch $dispatch_id | $count shifts | $who"
    fi
  done
done | sort
```

### 5. Get dispatch details

```bash
# View dispatch metadata
xbe view lineup-dispatches show <dispatch-id> --json

# Get all shifts in a specific dispatch
xbe view lineup-dispatch-shifts list --lineup-dispatch <dispatch-id> --json --limit 1000

# Count shifts per dispatch
xbe view lineup-dispatch-shifts list --lineup-dispatch <dispatch-id> --json | jq 'length'

# See who created the dispatch and when
xbe view lineup-dispatches show <dispatch-id> --fields created-by,created-at,status
```

## Common Dispatch Timing Patterns

Based on real-world data, dispatches typically follow this pattern:

**Evening before (9-11 PM)**:
- Bulk dispatch of 50+ shifts
- Gets most trucks assigned for next day
- Handled by evening dispatcher

**Late night (11 PM - 2 AM)**:
- Cleanup dispatches (1-6 shifts)
- Handles trucks that didn't accept initially
- Follow-up on rejected assignments

**Day-of (throughout work hours)**:
- Small dispatches (1-4 shifts each)
- Adjustments for schedule changes
- Replacement trucks needed
- Additional demand discovered

## Tracing Who Did What

```bash
# Find who created job production plans
xbe view job-production-plans show <job-id> --fields created-by,approved-by

# Find who created the lineup
xbe view lineups show <lineup-id> --fields created-by

# Find who dispatched shifts
xbe view lineup-dispatches show <dispatch-id> --fields created-by
```

## Working Around Data Model Limitations

**Limitation**: `lineup-dispatches` resource often has `null` for `lineup-id` and `created-by-id` in JSON output.

**Workaround**: 
1. Use `lineup-dispatch-shifts` to find dispatch IDs
2. Use table format (not JSON) to see created-by names
3. Use `created-at` filters on `lineup-dispatch-shifts` for timing analysis

```bash
# Table format shows created-by as name
xbe view lineup-dispatches show <dispatch-id> --fields created-by

# Or parse from table output
who=$(xbe view lineup-dispatches show <dispatch-id> --fields created-by 2>/dev/null | tail -1 | awk '{print $2, $3}')
```

## Example: Full Timeline Reconstruction

```bash
#!/bin/bash

LINEUP_ID=<lineup-id>
DATE=<date>

# 1. Get lineup details
echo "=== LINEUP DETAILS ==="
xbe view lineups show $LINEUP_ID --json

# 2. Count total shifts
echo "\n=== SHIFT COUNTS ==="
total=$(xbe view lineup-job-schedule-shifts list --lineup $LINEUP_ID --json --limit 1000 | jq 'length')
echo "Total shifts in lineup: $total"

# 3. Find all dispatch IDs
echo "\n=== FINDING DISPATCHES ==="
lineup_shift_ids=$(xbe view lineup-job-schedule-shifts list --lineup $LINEUP_ID --json --limit 1000 | jq -r '.[].id' | head -20)

dispatch_ids=$(for shift_id in $lineup_shift_ids; do
  xbe view lineup-dispatch-shifts list --lineup-job-schedule-shift $shift_id --json 2>/dev/null | jq -r '."lineup-dispatch-id"' 2>/dev/null
done | sort -u | grep -v null)

# 4. Analyze dispatch timing
echo "\n=== DISPATCH TIMELINE ==="
for hour in {00..23}; do
  for dispatch_id in $dispatch_ids; do
    count=$(xbe view lineup-dispatch-shifts list \
      --created-at-min "${DATE}T${hour}:00:00Z" \
      --created-at-max "${DATE}T${hour}:59:59Z" \
      --lineup-dispatch $dispatch_id \
      --json 2>/dev/null | jq 'length' 2>/dev/null)
    if [ "$count" != "0" ] && [ -n "$count" ]; then
      who=$(xbe view lineup-dispatches show $dispatch_id --fields created-by 2>/dev/null | tail -1 | awk '{print $2, $3}')
      echo "${DATE} ${hour}:xx | Dispatch $dispatch_id | $count shifts | $who"
    fi
  done
done | sort
```

## Key Insights

1. **Multiple dispatches are normal**: A single lineup may have 10+ separate dispatch events as adjustments are made
2. **Timing matters**: Bulk dispatching happens evening before, day-of dispatches are adjustments
3. **Work backward from shifts**: Use `lineup-dispatch-shifts` to find dispatches, not the other way around
4. **Created-at is your friend**: Use `created-at` filters on shifts to understand timing when dispatch metadata is limited
