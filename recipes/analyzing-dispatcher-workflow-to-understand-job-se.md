---
title: Analyzing dispatcher workflow to understand job selection criteria
when: When you need to understand how a dispatcher decides which jobs and shifts to include in a lineup, or when trying to identify the business logic/selection criteria behind lineup creation decisions
---

## Problem

You need to understand the dispatcher's decision-making process: which jobs get added to lineups, which shifts within those jobs are selected, and what criteria guide these choices.

## Approach

This is a multi-step investigative analysis that reveals both what the dispatcher did AND what domain knowledge is still hidden.

### Step 1: Identify the lineup and get basic scope

```bash
# Find lineups for the broker in the target date range
xbe view lineups list --broker <broker-id> --json --limit 100

# Get details on a specific lineup
xbe view lineups show <lineup-id> --json

# Count shifts in the lineup
xbe view lineup-job-schedule-shifts list --lineup <lineup-id> --json --limit 200 | jq 'length'
```

### Step 2: Trace which jobs were added to the lineup

```bash
# Get all lineup-job-production-plans for this lineup
xbe view lineup-job-production-plans list --lineup <lineup-id> --fields job-production-plan,created-by,created-at --json --limit 100

# Extract unique job production plan IDs
xbe view lineup-job-production-plans list --lineup <lineup-id> --json --limit 100 | jq -r '.[]."job-production-plan-id"' | sort -u

# Count how many jobs were added
xbe view lineup-job-production-plans list --lineup <lineup-id> --json --limit 100 | jq -r '.[]."job-production-plan-id"' | sort -u | wc -l
```

### Step 3: Understand the total universe of possible jobs

```bash
# Get all job production plans for the same date and broker
xbe view job-production-plans list --start-date <date> --broker <broker-id> --json --limit 500 | jq 'length'

# This reveals the selection ratio: X jobs chosen out of Y total
```

### Step 4: Analyze shift selection within chosen jobs

```bash
# For each job production plan, count total shifts
xbe view job-schedule-shifts list --start-date <date> --job-production-plan <job-production-plan-id> --json | jq 'length'

# Compare to how many shifts from that job made it into the lineup
xbe view lineup-job-schedule-shifts list --lineup <lineup-id> --json --limit 200 | jq '[.[] | select(."job-production-plan-id" == "<job-production-plan-id>")] | length'

# Calculate selection ratio for shifts
```

### Step 5: Look for selection criteria indicators

Check if there's a field that indicates "needs lineup dispatch":

```bash
# Check available fields on job-schedule-shifts
xbe view job-schedule-shifts show <shift-id> --json | jq 'keys'

# Test potential filtering fields like is-managed
xbe view job-schedule-shifts list --start-date <date> --broker <broker-id> --is-managed true --json --limit 500 | jq 'length'

# Compare managed shift count to lineup shift count
```

### Step 6: Analyze temporal patterns

Use the "Analyzing lineup shift creation patterns and dispatcher behavior" recipe to understand:
- How shifts were added over time (batch patterns)
- Whether there were multiple data entry sessions
- Company-by-company entry patterns

### Step 7: Identify knowledge gaps

Document what you CAN'T determine from the data:
- Why specific jobs were selected vs others
- Why specific shifts within a job were selected
- What business rules or customer contracts drive selection
- What constitutes "complete" for the dispatcher
- How exceptions are handled

## Key Insight

The data shows WHAT the dispatcher did but not WHY. Observable patterns include:
- Job-level filtering (some subset of available jobs)
- Shift-level filtering (some subset of shifts within chosen jobs)
- Temporal/batching patterns (how shifts are entered)
- Company grouping patterns (shifts grouped by trucker)

But the selection criteria remain implicit domain knowledge:
- Customer contract requirements
- Trucker type (brokered vs self-managed)
- Job priority or urgency
- Capacity constraints
- Relationship management factors

## Example Analysis Output

"For lineup 235396 on Oct 1, 2025:
- 11 job production plans added (out of 60 total jobs that day)
- 130 shifts added (out of 388 total shifts across those 11 jobs)
- 326 total managed shifts existed, but only 130 went into lineup
- Selection criteria unclear: no single field like is-managed fully explains the selection
- Dispatcher spent 1+ hour adding shifts in company-grouped batches
- Domain knowledge required: Why these 11 jobs? Why these 130 shifts?"

## Related Recipes

- "Analyzing lineup shift creation patterns and dispatcher behavior" - for temporal analysis
- "Tracing lineup shift batch creation and dispatch timeline patterns" - for understanding batch entry
- "Tracing the complete lifecycle of a lineup dispatch" - for end-to-end process understanding
