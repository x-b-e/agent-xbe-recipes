---
title: Understanding lineup-dispatch data model limitations
when: When you need to analyze who created lineup dispatches or trace dispatches back to their parent lineups, and you encounter null lineup-id or created-by-id fields
---

## Problem

The `lineup-dispatches` resource has significant data model limitations:
- The `lineup-id` field is consistently `null`, making it impossible to trace a dispatch back to its parent lineup
- The `created-by-id` field is consistently `null`, making it impossible to determine who created the dispatch
- These limitations appear to be systemic across all time periods

## What You Can Get from lineup-dispatches

You CAN see:
- Fulfillment results (`fulfillment-result`, `is-fulfilled`)
- Auto-offer settings (`auto-offer-trucker-tenders`, `auto-accept-trucker-tenders`)
- Fulfillment method (`fulfilled-with-auto-offer`)
- Fulfillment count

## What You Cannot Get

You CANNOT determine:
- Which lineup the dispatch belongs to
- Who created the dispatch
- Customer association (must be inferred through other means)

## Workaround Strategies

### If you need to associate dispatches with customers:
1. Query `lineup-dispatch-shifts` which have `job-schedule-shift-id`
2. From shifts, get `job-production-plan-id`
3. From job production plans, get `project-id` and then `customer-id`

```bash
# Get dispatch shifts
xbe view lineup-dispatch-shifts list --lineup-dispatch <dispatch-id> --json

# For each shift, get the job production plan
xbe view job-schedule-shifts show <shift-id> --json | jq -r '."job-production-plan-id"'

# Get the project/customer from the job production plan
xbe view job-production-plans show <plan-id> --json | jq '{"project-id", "customer-id"}'
```

### If you need to find who approved plans:
Do NOT rely on lineup-dispatches. Instead:
1. Query `job-production-plans` directly
2. Look at `job-production-plan-status-changes` for approval events
3. Check `job-production-plan-time-card-approvers` for designated approvers

## Example: Failed Approach

```bash
# This will show null values for lineup-id and created-by-id
xbe view lineup-dispatches list --created-at-min <date> --json | \
  jq -r '.[].id' | while read id; do \
    xbe view lineup-dispatches show $id --json | jq '{id, "lineup-id", "created-by-id"}'; \
  done

# Output:
# {"id":"<id>","lineup-id":null,"created-by-id":null}
```

This is a known limitation, not a data filtering issue.
