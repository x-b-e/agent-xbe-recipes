---
title: Tracing the complete lifecycle of a lineup dispatch
when: When you need to understand the full sequence of events from job planning through dispatch fulfillment for a lineup, or when you need to trace who did what and when in the lineup/dispatch process
---

## Overview

A lineup dispatch lifecycle involves multiple phases and resources. This recipe shows how to trace the complete flow from job planning through dispatch fulfillment.

## Data Model Relationships

```
job-production-plans
  └─> job-schedule-shifts (individual truck shifts for a job)
      └─> lineup-job-schedule-shifts (shifts added to a lineup)
          └─> lineup (groups shifts for a specific date/time window)
              └─> lineup-dispatches (dispatch actions)
                  └─> lineup-dispatch-shifts (individual shift assignments)
```

## Phase 1: Find the Lineup

```bash
# Find lineups by broker and get basic info
xbe view lineups list --broker <broker-id> --json --limit 100

# Get lineup details including time window
xbe view lineups show <lineup-id> --json
```

## Phase 2: Understand Shifts in the Lineup

```bash
# Count total shifts in lineup
xbe view lineup-job-schedule-shifts list --lineup <lineup-id> --json | jq 'length'

# Get sample shifts with assignments
xbe view lineup-job-schedule-shifts list --lineup <lineup-id> --limit 10 \
  --fields job-schedule-shift,trucker,trailer-type,is-ready-to-dispatch,start-at,end-at

# Get a specific shift's details
xbe view lineup-job-schedule-shifts show <lineup-job-schedule-shift-id> --json
```

## Phase 3: Trace Back to Job Plans

```bash
# Get job schedule shift details
xbe view job-schedule-shifts show <job-schedule-shift-id> --fields job,broker,customer,start-at

# Get the job production plan
xbe view job-production-plans show <job-id> \
  --fields name,customer,broker,planner,created-by,goal-tons,goal-hours,status
```

## Phase 4: Analyze Dispatch Events

```bash
# Find all dispatches for the lineup
xbe view lineup-dispatches list --lineup <lineup-id> --json

# Get detailed dispatch info including settings
xbe view lineup-dispatches show <lineup-dispatch-id> --json

# Trace through multiple dispatches
for dispatch_id in <dispatch-id-1> <dispatch-id-2> <dispatch-id-3>; do
  echo "=== Dispatch $dispatch_id ==="
  xbe view lineup-dispatches show $dispatch_id --fields created-by,is-fulfilled,fulfillment-result
  xbe view lineup-dispatch-shifts list --lineup-dispatch $dispatch_id --json | jq -r 'length' | \
    xargs -I {} echo "Shifts dispatched: {}"
  echo
done
```

## Phase 5: See Trucker Assignments

```bash
# Get shifts by trucker from a dispatch
xbe view lineup-dispatch-shifts list --lineup-dispatch <lineup-dispatch-id> \
  --fields trucker --json | jq -r 'group_by(."trucker-id") | 
  map({trucker: .[0].trucker, shifts: length})'

# Get driver assignments for fulfilled shifts
xbe view lineup-dispatch-shifts show <lineup-dispatch-shift-id> \
  --fields lineup-job-schedule-shift,driver,truck,trailer
```

## Common Patterns

### Multiple Dispatch Rounds

Lineups often require multiple dispatch actions:
- Initial bulk dispatch (e.g., 50 shifts)
- Follow-up dispatches for additions or changes (1-4 shifts each)
- Multiple dispatchers may be involved

### Dispatch Settings Impact

Key dispatch settings affect behavior:
- `auto-offer-to-customer-tenders`: Automatically create customer tenders
- `auto-offer-to-trucker-tenders`: Automatically send offers to truckers
- `auto-accept-trucker-tenders`: Automatically accept trucker responses

### Timeline Sequence

1. **Days/weeks before**: Job production plans created and approved
2. **Days before**: Job schedule shifts created for specific dates
3. **Day before/morning of**: Lineup created and shifts added
4. **Morning of**: First major dispatch
5. **Throughout day**: Additional dispatches as needed
6. **During day**: Drivers execute shifts, transactions recorded

## Tips

- Use `--json` with `jq` for complex aggregations across relationships
- Dispatches may have `null` for `lineup-id` due to data model limitations (see recipe: "Understanding lineup-dispatch data model limitations")
- Lineup filters don't support `start-at-min`/`start-at-max` despite help text suggesting they do
- Check `fulfillment-result` to verify dispatch success
- Different dispatchers can work on the same lineup over time
