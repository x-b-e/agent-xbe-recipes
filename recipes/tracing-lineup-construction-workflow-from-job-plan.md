---
title: Tracing lineup construction workflow from job plans to shift selection
when: When you need to understand how a lineup is built from scratch, including which job production plans are added, when shifts are selected, how truckers are assigned to shifts, and the overall dispatcher workflow for creating a lineup
---

# Tracing lineup construction workflow from job plans to shift selection

## Problem

You need to understand the complete workflow of how a dispatcher builds a lineup, from initial job plan selection through shift assignment and trucker allocation.

## Key Discovery: Lineup List Filter Limitations

The `lineup list` command does NOT support filtering by `start-at-min` or `start-at-max` despite these being documented in the help text. You'll get errors like:

```
{"errors":[{"title":"Filter not allowed","detail":"start_at_min is not allowed."}]}
```

Instead, filter by `--broker` or `--customer` and manually inspect the results.

## Overall Workflow Pattern

The lineup construction follows this sequence:

1. **Job Production Plans Added to Lineup** (rapid batch, ~3-4 seconds apart)
2. **Delay Period** (several minutes)
3. **Lineup Shifts Added** (extended period, grouped by trucking company)

## Step-by-Step Investigation

### 1. Find the Lineup

```bash
xbe view lineups list --broker <broker-id> --json --limit 100
```

Pick a lineup ID and get its details:

```bash
xbe view lineups show <lineup-id> --json
```

### 2. Identify Job Plans Added to Lineup

```bash
xbe view lineup-jobs list --lineup <lineup-id> --fields job-production-plan,created-at --json --limit 1000 | jq -r '.[] | "\(."created-at") - Job Plan \(."job-production-plan-id")"' | sort
```

This shows:
- Timing of when each job plan was added
- Batch pattern (typically all added within 1-2 minutes)
- Which job production plans are included

### 3. Analyze Shift Selection Timeline

```bash
xbe view lineup-job-schedule-shifts list --lineup-job <lineup-job-id> --fields created-at,accepted-trucker --json --limit 1000 | jq -r '.[] | "\(."created-at") - \(."accepted-trucker")"' | sort
```

Key patterns to look for:
- **Company grouping**: Shifts added in batches by trucking company
- **Time gaps**: Pauses between company batches suggest manual review/selection
- **Duration**: Total time from first shift to last shift added

### 4. Understand Pre-Existing Trucker Assignments

**Critical finding**: Trucker assignments already exist on job schedule shifts BEFORE they're added to the lineup.

Check when the underlying job schedule shift was created:

```bash
xbe view job-schedule-shifts show <shift-id> --fields created-at,accepted-trucker,job-production-plan --json
```

Compare this `created-at` to when the shift was added to the lineup. You'll typically find:
- Job schedule shifts created hours earlier (morning)
- Truckers already assigned at job shift creation time
- Lineup shifts created in afternoon by copying from job shifts

### 5. Check for Assignment Recommendations (Usually None)

```bash
xbe view lineup-job-schedule-shift-trucker-assignment-recommendations list --lineup-job-schedule-shift <shift-id> --fields candidates,created-at --json
```

Typically returns `[]`, indicating trucker assignments were NOT made through a recommendation system for this lineup.

## Interpretation: What the Dispatcher Sees

Based on the timing patterns, the dispatcher likely:

1. **Has a view of all job schedule shifts for tomorrow** that need lineup dispatch
2. **Shifts are already assigned to truckers** (done earlier by planners or scheduling system)
3. **Selects which jobs need coordinated dispatch** (adds job plans to lineup)
4. **Works through shifts company-by-company**:
   - Possibly has a list grouped/sorted by trucking company
   - Selects shifts to add to lineup (checkboxes or similar)
   - System copies trucker assignment from job shift to lineup shift
5. **This is selection work, not assignment work** - choosing which pre-assigned shifts go into THIS lineup

## Company-by-Company Pattern Example

If you see timing like:
```
2:44:01 PM - Knifong & Son Trucking
2:44:03 PM - Knifong & Son Trucking
2:44:05 PM - Knifong & Son Trucking
2:47:12 PM - Scot Heller Trucking
2:47:15 PM - Scot Heller Trucking
2:51:20 PM - AB Rock Co.
2:51:22 PM - AB Rock Co.
```

This indicates:
- Dispatcher working methodically through companies
- Time gaps = switching between companies or reviewing
- NOT doing capacity calculations (already done)
- NOT assigning truckers (already assigned)

## Key Data Model Insights

- **lineup-jobs**: Links job production plans to lineups
- **lineup-job-schedule-shifts**: Links job schedule shifts to lineup jobs
- **job-schedule-shifts**: Original shifts with trucker assignments (created earlier)
- **Trucker assignment flows**: Job Shift → Lineup Shift (copy), not Lineup → Assignment

## Common Mistakes to Avoid

1. **Don't assume lineup creation = trucker assignment**: Assignments happen earlier
2. **Don't use start-at filters on lineup list**: They're not supported
3. **Don't expect assignment recommendations**: Most lineups don't use them
4. **Don't skip checking job schedule shift creation time**: This reveals the pre-assignment timing

## Related Recipes

- "Understanding lineup list filter limitations" - Details on filter restrictions
- "Analyzing lineup shift creation patterns and dispatcher behavior" - Analyzing shift entry velocity
- "Tracing lineup dispatch timeline and dispatcher activity" - Understanding dispatch execution after lineup creation
