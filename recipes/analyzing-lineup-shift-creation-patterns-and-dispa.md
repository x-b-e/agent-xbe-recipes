---
title: Analyzing lineup shift creation patterns and dispatcher behavior
when: When you need to understand how shifts are being added to a lineup over time, identify data entry sessions and patterns, measure entry velocity, or analyze dispatcher workflow behavior (e.g., batch entry by company, data entry speed)
---

## Goal
Analyze how lineup shifts are being created over time to understand dispatcher behavior, identify data entry sessions, measure entry velocity, and detect patterns like batch entry by trucking company.

## Prerequisites
- Lineup ID
- Understanding that lineup shifts are the individual assignments within a lineup

## Steps

### 1. Get all lineup shifts with timestamps
```bash
xbe view lineup-shifts list \
  --lineup <lineup-id> \
  --fields trucker,created-at \
  --json \
  --limit 1000
```

### 2. Sort by creation timestamp to see chronological order
```bash
xbe view lineup-shifts list \
  --lineup <lineup-id> \
  --fields trucker,created-at \
  --json \
  --limit 1000 | \
  jq 'sort_by(."created-at")'
```

### 3. Analyze entry patterns

Look for:
- **Batch entry by company**: Multiple shifts for the same trucker created within seconds
- **Entry sessions**: Gaps of several minutes between groups of shifts
- **Entry velocity**: Number of shifts created per minute
- **Total timespan**: Time from first shift to last shift

### 4. Create a visual timeline

Format the data to show temporal patterns:

```bash
xbe view lineup-shifts list \
  --lineup <lineup-id> \
  --fields trucker,created-at \
  --json \
  --limit 1000 | \
  jq -r 'sort_by(."created-at")[] | "\(.id)  \(."created-at")  \(.trucker."company-name")"'
```

## Key Patterns to Identify

### Batch Entry by Company
When multiple shifts for the same trucker are created in rapid succession (0.2-2 seconds apart), this indicates the dispatcher is working through their list company-by-company.

Example output showing batch entry:
```
<shift-id>  2025-09-30T19:44:59.4437Z  <trucker-name>
<shift-id>  2025-09-30T19:44:59.7099Z  <trucker-name>
<shift-id>  2025-09-30T19:44:59.9369Z  <trucker-name>
<shift-id>  2025-09-30T19:45:00.1815Z  <trucker-name>
```

### Entry Sessions
Gaps of several minutes (5+ minutes) between shifts indicate the dispatcher took a break or switched tasks. These gaps help identify distinct data entry sessions.

### Entry Velocity
Calculate shifts per minute within each session:
- Slow entry: 1-3 shifts/minute (careful manual entry)
- Moderate entry: 4-7 shifts/minute (experienced dispatcher)
- Fast entry: 8+ shifts/minute (possible bulk import or copy/paste)

## Analysis Output Example

Create a summary showing:
- **Session 1**: 2:44-2:55 PM → 57 shifts in 11 minutes (5 shifts/min)
- **Session 2**: 3:05-3:12 PM → 22 shifts in 7 minutes (3 shifts/min)
- **Session 3**: 3:56-3:59 PM → 28 shifts in 3 minutes (9 shifts/min)
- **Total**: 107 shifts over 1 hour 15 minutes of active entry
- **Gap before dispatch**: 12 minutes between last shift and dispatch event

## Related Recipes
- "Tracing lineup dispatch timeline and dispatcher activity" - for analyzing when dispatches occurred
- "Tracing individual lineup shift lifecycle from creation to fulfillment" - for following a single shift
- "Tracing the complete lifecycle of a lineup dispatch" - for understanding the full dispatch process

## Notes
- Creation timestamps are in UTC with millisecond precision
- Batch entry patterns (multiple shifts created within seconds) suggest manual entry, not API/bulk import
- The dispatcher's workflow often groups shifts by trucking company
- Gaps between entry sessions may indicate coordination with other teams or waiting for information
