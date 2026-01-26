---
title: Analyzing dispatcher workflow to understand job selection and trucker assignment
when: When you need to understand how a dispatcher decides which jobs and shifts to include in a lineup, how truckers are assigned to lineup shifts, or when trying to identify the business logic/selection criteria behind lineup creation and dispatch decisions
---

# Analyzing dispatcher workflow to understand job selection and trucker assignment

## Context
Dispatchers create lineups by selecting jobs and shifts, then assigning truckers to those shifts before dispatching. Understanding this workflow requires tracing through multiple resources and understanding the relationship between job-schedule-shifts and lineup-job-schedule-shifts.

## Key Concepts

### Job Schedule Shift vs Lineup Job Schedule Shift
- **job-schedule-shift**: Represents a shift that needs to be worked (created earlier, may have accepted-trucker through tender system)
- **lineup-job-schedule-shift**: Represents that same shift AS PART OF A LINEUP, with its own `trucker` field that the dispatcher sets
- The trucker assignment decision happens on the **lineup-job-schedule-shift**, not the job-schedule-shift

## Complete Dispatcher Workflow

### Step 1: Add Jobs to Lineup
```bash
# Add job production plan to lineup
xbe do lineup-job-production-plans create \
  --lineup <lineup-id> \
  --job-production-plan <job-production-plan-id>
```

### Step 2: Create Lineup Shifts (Without Trucker)
```bash
# Create lineup shift (no trucker assignment yet)
lineup_shift_id=$(xbe do lineup-job-schedule-shifts create \
  --lineup <lineup-id> \
  --job-schedule-shift <job-schedule-shift-id> \
  --is-ready-to-dispatch false \
  --json | jq -r '.id')
```

### Step 3: Assign Trucker to Lineup Shift
```bash
# Update lineup shift with trucker assignment
xbe do lineup-job-schedule-shifts update <lineup-shift-id> \
  --trucker <trucker-id> \
  --is-ready-to-dispatch true
```

### Step 4: Dispatch the Lineup
```bash
# Dispatch all ready shifts
xbe do lineup-dispatches create --lineup <lineup-id> \
  --auto-offer-trucker-tenders true \
  --auto-offer-customer-tenders true
```

## Analyzing Selection Criteria

To understand WHY a dispatcher chose specific jobs/shifts/truckers:

### Find What Jobs Were Added
```bash
# Get all job production plans in a lineup
xbe view lineup-job-production-plans list \
  --lineup <lineup-id> \
  --json
```

### Find What Shifts Were Created
```bash
# Get all lineup shifts with trucker assignments
xbe view lineup-job-schedule-shifts list \
  --lineup <lineup-id> \
  --fields trucker,job-schedule-shift,is-ready-to-dispatch \
  --json
```

### Analyze Creation Timeline
```bash
# See when shifts were added (batch patterns)
xbe view lineup-job-schedule-shifts list \
  --lineup <lineup-id> \
  --fields created-at,trucker,job-schedule-shift \
  --json | jq -r '.[] | "\(."created-at") - Trucker: \(.trucker."company-name" // "none") - Shift: \(."job-schedule-shift-id")"'
```

### Find Selection Patterns
```bash
# Group by trucker to see if dispatcher assigned company-by-company
xbe view lineup-job-schedule-shifts list \
  --lineup <lineup-id> \
  --json | jq 'group_by(."trucker-id") | map({trucker: .[0].trucker."company-name", count: length})'
```

## Common Patterns

### Company-by-Company Assignment
Dispatchers often assign all shifts for one trucker before moving to the next:
```bash
# Look for temporal clustering by trucker
xbe view lineup-job-schedule-shifts list \
  --lineup <lineup-id> \
  --sort created-at \
  --fields created-at,trucker \
  --json
```

### Job-by-Job Assignment
Or they might assign all shifts for one job before moving to the next:
```bash
# Group by job to see job-level patterns
xbe view lineup-job-schedule-shifts list \
  --lineup <lineup-id> \
  --json | jq 'group_by(."job-schedule-shift"."job-id") | map({job: .[0]."job-schedule-shift".job.name, shifts: length})'
```

## Integration with ML Recommendations

See the "Using ML-powered trucker assignment recommendations for lineup shifts" recipe for how to integrate machine learning recommendations into Step 3 above.

## Troubleshooting

### Trucker Shows on Shift But Can't Be Set
If `xbe view job-schedule-shifts` shows a trucker but you can't set it with `--trucker`, that's because:
- The trucker on job-schedule-shift comes from `accepted-broker-tender` (tender system)
- To assign for lineup purposes, you set `--trucker` on the **lineup-job-schedule-shift**
- These are separate fields serving different purposes

### Understanding the Two Trucker Fields
- `job-schedule-shift.accepted-trucker`: Set through tender acceptance system
- `lineup-job-schedule-shift.trucker`: Set by dispatcher for lineup dispatch
- They may or may not be the same trucker
