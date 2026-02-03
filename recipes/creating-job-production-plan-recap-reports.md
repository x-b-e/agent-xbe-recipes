---
title: Creating job production plan recap reports
when: When you need to create a comprehensive report showing actual performance vs planned performance across multiple job production plans, including timing issues, quantity variances, incidents, and key concerns
---

# Creating job production plan recap reports

To create a comprehensive report analyzing multiple job production plans with actual vs planned performance:

## Step 1: Get the list of plans

First, retrieve the list of job production plans for the target date:

```bash
xbe view job-production-plans list --start-on <date> --customer <customer-id-1>,<customer-id-2>
```

This returns a table with plan IDs, status, dates, and basic metrics.

## Step 2: Retrieve detailed recaps for each plan

For each plan ID, fetch the detailed recap in parallel:

```bash
# Multiple parallel calls
xbe view job-production-plans recap <plan-id-1> --json
xbe view job-production-plans recap <plan-id-2> --json
xbe view job-production-plans recap <plan-id-3> --json
# ... etc
```

The recap provides comprehensive details including:
- Planned vs actual timing (start/end times, duration)
- Material quantities (planned vs actual, variances)
- Rate per hour comparisons
- Material sites and distances
- All incidents (production, efficiency, administrative, liability, safety)
- Trucking metrics (GPS visibility, surplus percentages, dispatches)
- Status change history

## Step 3: Synthesize the report

Analyze the recap data to identify:

**Timing Issues:**
- Early/late starts (calculate minute variance)
- Duration variance (planned vs actual hours)

**Quantity Performance:**
- Significant shortfalls (e.g., >50% under plan)
- Over-deliveries (exceeding plan)
- Zero production jobs (100% shortfall)
- Material-specific misses (e.g., no millings delivered when planned)

**Efficiency Metrics:**
- Rate per hour variance (% difference from plan)
- Surplus percentage changes (planned vs actual)
- GPS visibility percentages

**Incidents:**
- Production incidents (delays, timing issues)
- Trucking incidents (transaction order issues)
- Other incident types

**Key Concerns:**
- Patterns across multiple jobs (e.g., repeated material type shortfalls)
- Low GPS visibility
- Time card approval status
- Critical timing or quantity variances

## Report Structure

Organize the synthesized information as:

1. **Summary**: Total plans, completion status overview
2. **Most Notable Items**: Top 4-6 jobs with significant variances or issues
3. **Zero Production Jobs**: List of jobs with no actual deliveries
4. **Key Concerns**: System-wide patterns and issues requiring attention

For each notable job, include:
- Job name and plan ID
- Timing variance with specific minutes early/late
- Quantity variance with percentages
- Key incidents
- GPS visibility
- Material-specific issues
