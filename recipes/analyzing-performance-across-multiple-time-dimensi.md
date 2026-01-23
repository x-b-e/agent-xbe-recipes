---
title: Analyzing performance across multiple time dimensions (overall, monthly, weekly)
when: When you need to break down performance data into multiple time granularities (overall summary, monthly trends, weekly trends) to identify patterns or determine if recent performance deviates from historical norms
---

# Analyzing performance across multiple time dimensions (overall, monthly, weekly)

## Pattern

When analyzing whether recent performance is unusual, gather historical data at multiple time granularities:
1. Overall aggregate (entire period)
2. Monthly breakdowns (to see seasonal patterns)
3. Weekly breakdowns (to see recent trends)
4. By-entity breakdowns (to compare different customers/truckers/etc.)

## Example: Job Production Plan Analysis

### 1. Get historical data for a date range

```bash
# Get 3 months of historical data
xbe view job-production-plans list \
  --start-on-min <start-date> \
  --start-on-max <end-date> \
  --customer <customer-id-1>,<customer-id-2>,<customer-id-3> \
  --json > historical_plans.json
```

### 2. Calculate overall aggregate metrics

```bash
# Overall summary across all plans
cat historical_plans.json | jq '
  {
    total_plans: length,
    total_goal_tons: [.[].goal_quantity_tons] | add,
    total_actual_tons: [.[].actual_quantity_tons] | add,
    avg_achievement_pct: ([.[].actual_quantity_tons] | add) / ([.[].goal_quantity_tons] | add) * 100,
    avg_start_offset_min: [.[].start_time_offset_minutes] | add / length,
    avg_actual_surplus_pct: [.[].actual_surplus_percentage] | add / length
  }
'
```

### 3. Group by month for trend analysis

```bash
# Monthly breakdown
cat historical_plans.json | jq '
  group_by(.start_on[0:7]) | map({
    month: .[0].start_on[0:7],
    plans: length,
    goal_tons: [.[].goal_quantity_tons] | add,
    actual_tons: [.[].actual_quantity_tons] | add,
    achievement_pct: (([.[].actual_quantity_tons] | add) / ([.[].goal_quantity_tons] | add) * 100),
    avg_start_offset: [.[].start_time_offset_minutes] | add / length,
    avg_surplus_pct: [.[].actual_surplus_percentage] | add / length
  })
'
```

### 4. Group by week for recent trends

```bash
# Weekly breakdown (using ISO week)
cat historical_plans.json | jq '
  group_by(.start_on | split("-") | "\(.[0])-W\((.[1]|tonumber / 7 | floor))") | map({
    week: .[0].start_on,
    plans: length,
    goal_tons: [.[].goal_quantity_tons] | add,
    actual_tons: [.[].actual_quantity_tons] | add,
    achievement_pct: (([.[].actual_quantity_tons] | add) / ([.[].goal_quantity_tons] | add) * 100),
    avg_start_offset: [.[].start_time_offset_minutes] | add / length
  })
'
```

### 5. Break down by entity (customer/trucker/etc.)

```bash
# By customer
cat historical_plans.json | jq '
  group_by(.customer.id) | map({
    customer_id: .[0].customer.id,
    customer_name: .[0].customer.name,
    plans: length,
    goal_tons: [.[].goal_quantity_tons] | add,
    actual_tons: [.[].actual_quantity_tons] | add,
    achievement_pct: (([.[].actual_quantity_tons] | add) / ([.[].goal_quantity_tons] | add) * 100),
    avg_surplus_pct: [.[].actual_surplus_percentage] | add / length
  })
'
```

## Analysis Approach

1. **Overall baseline**: Establish normal ranges from aggregate metrics
2. **Monthly trends**: Identify seasonal patterns or month-to-month variations
3. **Weekly trends**: Detect recent deterioration or improvement
4. **Entity comparison**: See if issues are isolated to specific customers/truckers
5. **Compare current to baseline**: Determine if recent performance falls within historical norms

## Key Insight

This multi-dimensional approach helps distinguish between:
- **Systemic issues** (consistent across all time periods)
- **Seasonal patterns** (monthly variations)
- **Recent trends** (weekly deterioration/improvement)
- **Entity-specific problems** (isolated to certain customers/truckers)
