---
title: Understanding lineup list filter limitations
when: When you need to filter lineups by date range and encounter 'Filter not allowed' errors for start-at-min or start-at-max
---

# Understanding lineup list filter limitations

## The Problem

The `xbe view lineups list` command's help text mentions `--start-at-min` and `--start-at-max` filters:

```
Filters:
  --start-at-min Filter by start time on/after (ISO 8601)
  --start-at-max Filter by start time on/before (ISO 8601)
```

However, attempting to use these filters results in errors:

```bash
# This FAILS despite being in the help text
xbe view lineups list --broker <broker-id> \
  --start-at-min "<date>T00:00:00Z" \
  --start-at-max "<date>T23:59:59Z" \
  --json

# Error:
# {"errors":[{"title":"Filter not allowed",
#  "detail":"start_at_min is not allowed.",
#  "code":"FILTER_NOT_ALLOWED","status":"400"}]}
```

## Available Filters

The lineup list command only supports these filters:

```bash
# Supported filters
xbe view lineups list --broker <broker-id>      # Filter by broker
xbe view lineups list --customer <customer-id>  # Filter by customer
xbe view lineups list --name-like "<pattern>"   # Filter by name pattern
```

## Workarounds

### Option 1: Get all lineups and filter in post-processing

```bash
# Get all lineups for a broker
xbe view lineups list --broker <broker-id> --json --limit 1000 > lineups.json

# Filter by date using jq
jq '[.[] | select(
  (."start-at-min" >= "<start-date>T00:00:00Z") and 
  (."start-at-min" <= "<end-date>T23:59:59Z")
)]' lineups.json
```

### Option 2: Use lineup-job-schedule-shifts with date filters

If you're trying to find lineups active on a specific date, work backward from shifts:

```bash
# Find shifts created on a specific date
xbe view lineup-job-schedule-shifts list \
  --created-at-min "<date>T00:00:00Z" \
  --created-at-max "<date>T23:59:59Z" \
  --json --limit 1000 | jq -r '.[]."lineup-id"' | sort -u

# Then get details for those lineups
for lineup_id in <lineup-ids>; do
  xbe view lineups show $lineup_id --json
done
```

### Option 3: Use lineup-dispatch-shifts with date filters

If you're looking for lineups dispatched on a specific date:

```bash
# Find dispatch shifts created on a date
xbe view lineup-dispatch-shifts list \
  --created-at-min "<date>T00:00:00Z" \
  --created-at-max "<date>T23:59:59Z" \
  --json --limit 1000

# Extract lineup IDs from the shifts
jq -r '.[]."lineup-job-schedule-shift-id"' | \
  while read shift_id; do
    xbe view lineup-job-schedule-shifts show $shift_id --json | \
      jq -r '."lineup-id"'
  done | sort -u
```

## Why This Happens

The API may not support these filters even though the CLI help text includes them. This could be:
- A documentation bug (help text is incorrect)
- An API limitation (filters not implemented)
- A performance optimization (date filtering disabled for large datasets)

## Key Takeaway

When filtering lineups by date:
1. **Don't rely on start-at-min/max filters** - they don't work
2. **Fetch all lineups for the broker** and filter client-side with jq
3. **Or work backward** from lineup-job-schedule-shifts or lineup-dispatch-shifts which DO support date filters
