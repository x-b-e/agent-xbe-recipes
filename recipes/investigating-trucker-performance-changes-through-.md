---
title: Investigating trucker performance changes through historical transition analysis
when: When you need to understand why a trucker's performance (cost per ton, productivity) changed dramatically over time, especially when initial analysis of recent periods shows anomalies that might be explained by operational transitions, customer changes, or contract modifications
---

## Problem

When analyzing trucker performance over a recent period (e.g., 2-3 months) shows concerning metrics, you need to determine whether this represents:
- A persistent problem requiring action
- A transition period that's already improving
- Different types of work with different customers
- Contract or operational changes

## Solution

Perform a full historical analysis to identify transitions and trends:

### Step 1: Determine Available Historical Range

```bash
# Find when the trucker first started working
xbe view material-transactions lane-summary \
  --start-on-min <early-date> \
  --start-on-max <recent-date> \
  --trucker <trucker-id> \
  --broker <broker-id> \
  --group-by month \
  --json
```

This shows which months have data and helps you identify the full date range to analyze.

### Step 2: Pull Complete Monthly Breakdown

```bash
# Get month-by-month performance with customer breakdown
xbe view material-transactions lane-summary \
  --start-on-min <first-date-with-data> \
  --start-on-max <last-date-with-data> \
  --trucker <trucker-id> \
  --broker <broker-id> \
  --group-by month,customer \
  --include-effective-cost-per-hour \
  --json > <trucker-name>_monthly_history.json
```

Key metrics to extract for each month:
- Cost per ton (total_cost / tons)
- Tons per shift (tons / total_shifts)
- Trips per shift (trips / total_shifts)
- Customer name (to identify transitions)
- Effective cost per hour (if available)

### Step 3: Identify Operational Transitions

Look for:
- **Customer changes**: Did the trucker switch from one customer to another?
- **Performance jumps**: Month-over-month changes >50% in key metrics
- **Volume patterns**: Sudden increases/decreases in shift count or tonnage
- **Extreme values**: Months with unusually high/low cost per ton that might indicate different work types

### Step 4: Analyze Transition Periods

When you find a transition (e.g., customer change in October):

```bash
# Compare before vs after transition
# Period BEFORE transition
xbe view material-transactions lane-summary \
  --start-on-min <pre-transition-start> \
  --start-on-max <pre-transition-end> \
  --trucker <trucker-id> \
  --broker <broker-id> \
  --customer <old-customer-id> \
  --include-effective-cost-per-hour \
  --json

# Period AFTER transition  
xbe view material-transactions lane-summary \
  --start-on-min <post-transition-start> \
  --start-on-max <post-transition-end> \
  --trucker <trucker-id> \
  --broker <broker-id> \
  --customer <new-customer-id> \
  --include-effective-cost-per-hour \
  --json
```

Calculate improvement/degradation:
- Cost per ton change: (after - before) / before
- Productivity change: (after - before) / before
- Volume change: (after - before) / before

### Step 5: Check for Continuous Improvement/Degradation

If performance is changing month-over-month, check if it's trending:

```bash
# Get each month individually after transition
xbe view material-transactions lane-summary \
  --start-on <month-1> \
  --trucker <trucker-id> \
  --broker <broker-id> \
  --json

xbe view material-transactions lane-summary \
  --start-on <month-2> \
  --trucker <trucker-id> \
  --broker <broker-id> \
  --json

# etc. for each month
```

Look for:
- Consistent month-over-month improvement (learning curve)
- Consistent degradation (deteriorating relationship)
- Stabilization at a new level

### Step 6: Verify Current State

Check the most recent complete month to determine if the trend has continued:

```bash
xbe view material-transactions lane-summary \
  --start-on <most-recent-month> \
  --trucker <trucker-id> \
  --broker <broker-id> \
  --include-effective-cost-per-hour \
  --json
```

This tells you whether the trucker is:
- Still improving (problem solving itself)
- Stable at acceptable level (problem solved)
- Stable at unacceptable level (action needed)
- Degrading (urgent action needed)

## Key Insights to Extract

1. **Was there a transition?** Customer change, contract change, operational change
2. **How dramatic?** Calculate % improvement/degradation in key metrics
3. **What's the trend?** Improving, degrading, or stable
4. **What's the current state?** Most recent month's performance
5. **Is action needed?** Based on current state vs historical baseline

## Example Analysis Pattern

From the transcript:
- Oct-Nov analysis showed $51.70/ton (concerning)
- Full history revealed:
  - Jan-Sep: $217/ton with Customer A (terrible)
  - Oct: $79/ton with Customer B (improving)
  - Nov: $40/ton with Customer B (much better)
  - Dec: $24/ton with Customer B (market rate)
- **Conclusion**: Transition period, problem self-correcting, current state acceptable

Without the full historical view, the Oct-Nov analysis would have led to wrong conclusions.

## Related Recipes

- Use **investigating-trucking-cost-anomalies-by-analyzing-material-composition** to check if performance differences are due to different materials/work types
- Use **comparing-trucker-spend-and-efficiency-between-customers** to benchmark the trucker's performance against alternatives
