---
title: Analyzing historical performance baselines with multi-dimensional summaries
when: When you need to determine if current performance is unusual by comparing against historical patterns, or when you need to analyze trends across multiple time dimensions (overall, monthly, weekly)
---

# Analyzing Historical Performance Baselines

When users ask if current performance is "unusual" or "concerning", you need to compare against historical baselines. Use multi-dimensional job-production-plan-summary analysis.

## Pattern: Three-Level Temporal Analysis

### 1. Overall Baseline (All Data)
Get aggregate metrics across the entire period:

```bash
xbe summarize job-production-plan-summary create \
  --start-on <start-date> \
  --end-on <end-date> \
  --filter customer=<customer-id> \
  --filter status=approved \
  --metrics plan_count,goal_tons_sum,tons_sum,tons_vs_goal_tons_avg,start_time_offset_minutes_mean,actual_practical_surplus_pct,truck_hours_sum
```

This provides your baseline averages (e.g., "53% achievement is normal").

### 2. Monthly Trends
Group by month to see seasonal patterns and changes over time:

```bash
xbe summarize job-production-plan-summary create \
  --start-on <start-date> \
  --end-on <end-date> \
  --group-by month \
  --filter customer=<customer-id> \
  --filter status=approved \
  --metrics plan_count,goal_tons_sum,tons_sum,tons_vs_goal_tons_avg,start_time_offset_minutes_mean,actual_practical_surplus_pct
```

This shows whether recent performance represents a trend or anomaly.

### 3. Weekly Granularity (Recent Period)
For recent weeks, get detailed week-by-week breakdown:

```bash
xbe summarize job-production-plan-summary create \
  --start-on <recent-start-date> \
  --end-on <end-date> \
  --group-by week \
  --filter customer=<customer-id> \
  --filter status=approved \
  --metrics plan_count,goal_tons_sum,tons_sum,start_time_offset_minutes_mean,tons_vs_goal_tons_avg,production_incident_net_impact_minutes_sum,actual_practical_surplus_pct,truck_hours_sum
```

This reveals immediate trends ("start times worsening in week 4").

### 4. Customer-Level Breakdown
Group by customer to compare performance across entities:

```bash
xbe summarize job-production-plan-summary create \
  --start-on <start-date> \
  --end-on <end-date> \
  --group-by customer \
  --filter customer=<customer-id-1>,<customer-id-2> \
  --filter status=approved \
  --metrics plan_count,goal_tons_sum,tons_sum,tons_vs_goal_tons_avg,actual_practical_surplus_pct,start_time_offset_minutes_mean
```

## Key Metrics for Baseline Analysis

**Core Performance:**
- `plan_count` - Volume of activity
- `goal_tons_sum` - Planned tonnage
- `tons_sum` - Actual tonnage delivered
- `tons_vs_goal_tons_avg` - Achievement rate (0.53 = 53%)

**Operational Efficiency:**
- `start_time_offset_minutes_mean` - Average late/early starts
- `actual_practical_surplus_pct` - Excess capacity percentage
- `truck_hours_sum` - Total truck hours utilized

**Issues/Incidents:**
- `production_incident_net_impact_minutes_sum` - Total incident delays

## Interpretation Pattern

Compare the current observation against:
1. **Overall average** - Is it within normal range?
2. **Monthly trend** - Is this a new pattern or ongoing?
3. **Recent weeks** - Is it getting better or worse?
4. **Peer comparison** - Do other customers show the same pattern?

## Example Analysis Flow

```bash
# Step 1: Get 3-month baseline
xbe summarize job-production-plan-summary create \
  --start-on <3-months-ago> --end-on <today> \
  --filter customer=<customer-id> --filter status=approved \
  --metrics tons_vs_goal_tons_avg,start_time_offset_minutes_mean

# Result: 53% achievement, 67 min late average

# Step 2: Monthly breakdown to see trends
xbe summarize job-production-plan-summary create \
  --start-on <3-months-ago> --end-on <today> \
  --group-by month \
  --filter customer=<customer-id> --filter status=approved \
  --metrics tons_vs_goal_tons_avg,start_time_offset_minutes_mean

# Result: Oct 42%, Nov 57%, Dec 57%, Jan 43% - fluctuating but within range

# Step 3: Recent weekly detail
xbe summarize job-production-plan-summary create \
  --start-on <current-month-start> --end-on <today> \
  --group-by week \
  --filter customer=<customer-id> --filter status=approved \
  --metrics tons_vs_goal_tons_avg,start_time_offset_minutes_mean

# Result: Week 2: 43%, Week 3: 47%, Week 4: 36% - declining trend
```

## Answering "Is This Unusual?"

- **YES** if current values are >2x the historical average or outside historical range
- **NO** if current values fall within the historical range, even if undesirable
- **TRENDING** if recent weeks show consistent direction away from baseline

The key insight: Performance can be consistently poor and still be "normal" - unusual means statistically different from baseline, not subjectively bad.
