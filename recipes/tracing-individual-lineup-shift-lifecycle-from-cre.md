---
title: Tracing individual lineup shift lifecycle from creation to fulfillment
when: When you need to understand the complete timeline of a single shift from initial creation through trucker assignment, dispatch event, and final fulfillment with precise timestamps at each stage
---

# Tracing Individual Lineup Shift Lifecycle from Creation to Fulfillment

## Overview

This recipe shows how to trace a single shift through its complete lifecycle with precise timestamps:
1. **Creation**: lineup-job-schedule-shift created (shift added to lineup, trucker pre-assigned)
2. **Dispatch**: lineup-dispatch-shift created (tender sent to trucker)
3. **Fulfillment**: lineup-dispatch-shift updated (trucker confirms, driver assigned)

## Data Model

```
lineup-job-schedule-shift (the shift on the lineup)
  ↓
lineup-dispatch-shift (the dispatch event for that shift)
  ↓ (links to)
lineup-dispatch (the batch dispatch event)
```

## Step 1: Get the Lineup Shift Details

Start with a lineup-job-schedule-shift ID and get its creation and update timestamps:

```bash
xbe view lineup-job-schedule-shifts show <shift-id> \
  --fields created-at,updated-at,trucker,job-schedule-shift
```

Key fields:
- `created-at`: When the shift was added to the lineup and trucker was pre-assigned
- `updated-at`: When the shift was last modified (often when it was fulfilled)
- `trucker`: Which trucking company was assigned

## Step 2: Find the Dispatch Event

Get the lineup-dispatch-shift that corresponds to this lineup shift:

```bash
xbe view lineup-dispatch-shifts list \
  --lineup-job-schedule-shift <shift-id> \
  --fields created-at,updated-at,fulfilled-trucker,lineup-dispatch
```

Key fields:
- `created-at`: When the dispatch was sent (when dispatcher clicked "Dispatch")
- `updated-at`: When the trucker confirmed/fulfilled
- `fulfilled-trucker`: Which trucker fulfilled it (should match assignment)
- `lineup-dispatch`: The batch dispatch event ID

## Step 3: Get the Dispatcher Information

Use the lineup-dispatch ID from step 2:

```bash
xbe view lineup-dispatches show <dispatch-id> \
  --fields created-at,created-by
```

Key fields:
- `created-at`: When the dispatcher initiated the batch dispatch
- `created-by`: Who clicked the "Dispatch" button

## Step 4: Calculate Time Gaps

Compare the timestamps to understand timing:

1. **Creation to Dispatch**: Time between lineup shift creation and dispatch
   - Large gap (hours/days): Pre-planned shift
   - Small gap (seconds): Just-in-time shift added right before dispatch

2. **Dispatch to Fulfillment**: Time for trucker to accept
   - Auto-accept enabled: Milliseconds to a few seconds
   - Manual acceptance: Minutes to hours

## Complete Example

```bash
# Step 1: Get shift details
xbe view lineup-job-schedule-shifts show <shift-id> \
  --fields created-at,updated-at,trucker,job-schedule-shift

# Output shows:
# created-at: 2025-09-30T20:59:58.7790Z (shift added to lineup)
# updated-at: 2025-09-30T21:11:59.5844Z (shift fulfilled)

# Step 2: Find dispatch event
xbe view lineup-dispatch-shifts list \
  --lineup-job-schedule-shift <shift-id> \
  --fields created-at,updated-at,fulfilled-trucker,lineup-dispatch

# Output shows:
# created-at: 2025-09-30T21:11:54.7779Z (dispatch sent)
# updated-at: 2025-09-30T21:11:59.5713Z (trucker confirmed)
# lineup-dispatch: <dispatch-id>

# Step 3: Get dispatcher
xbe view lineup-dispatches show <dispatch-id> \
  --fields created-at,created-by

# Output shows:
# created-at: 2025-09-30T21:11:54.7748Z (dispatcher clicked)
# created-by: Jeremy Davis
```

## Timing Analysis

From the example above:

1. **Shift Created**: 20:59:58.779
2. **Dispatch Initiated**: 21:11:54.774 (11 minutes 56 seconds later)
3. **Dispatch Sent to Shift**: 21:11:54.777 (3 milliseconds later)
4. **Trucker Fulfilled**: 21:11:59.571 (4.8 seconds after dispatch)

This shows:
- Pre-planned: Shift created ~12 minutes before dispatch
- Instant dispatch: System created dispatch-shift within 3ms of batch dispatch
- Fast fulfillment: Auto-accept enabled, confirmed in under 5 seconds

## Alternative: Combined Query

You can combine all steps into a single bash command:

```bash
echo "=== Lineup Shift Timeline ===" && \
xbe view lineup-job-schedule-shifts show <shift-id> \
  --fields created-at,updated-at,trucker,job-schedule-shift && \
echo && echo "=== Dispatch Shift ===" && \
xbe view lineup-dispatch-shifts list \
  --lineup-job-schedule-shift <shift-id> \
  --fields created-at,updated-at,fulfilled-trucker,lineup-dispatch && \
echo && echo "=== Dispatch Event ===" && \
xbe view lineup-dispatches show <dispatch-id> \
  --fields created-at,created-by
```

## Common Patterns

**Pre-planned shifts (night before):**
- Created: Hours before dispatch
- Dispatched: Bulk batch (50+ shifts at once)
- Fulfilled: Seconds (auto-accept)
- Gap: 15-87 minutes typical

**Just-in-time shifts (day of):**
- Created: Seconds before dispatch
- Dispatched: Small batch (1-5 shifts)
- Fulfilled: Milliseconds (immediate auto-accept)
- Gap: 7-30 seconds typical

## Related Recipes

- Use "Tracing lineup dispatch timeline and dispatcher activity" to analyze all dispatches for a lineup
- Use "Tracing the complete lifecycle of a lineup dispatch" to trace from job planning through dispatch
- Use "Analyzing lineup dispatch patterns by creator" to analyze dispatch patterns by who created them
