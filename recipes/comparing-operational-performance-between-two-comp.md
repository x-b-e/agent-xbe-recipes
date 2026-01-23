---
title: Comparing operational performance between two companies
when: When you need to compare operational performance metrics between two companies or brokers across multiple dimensions (tonnage achievement, on-time performance, cost efficiency, trends over time, etc.) to identify competitive advantages or operational gaps
---

# Comparing Operational Performance Between Two Companies

This recipe shows how to conduct a comprehensive performance comparison between two companies using job production plan data.

## Overview

The workflow involves:
1. Identifying all related customer entities for each company
2. Gathering job production plan data for a comparison period
3. Aggregating metrics across multiple dimensions
4. Analyzing trends and patterns
5. Creating a comprehensive comparison report

## Step 1: Identify Customer Entities

Both companies may have multiple customer entities (e.g., Construction, Materials, Transport divisions).

```bash
# Find all customer entities for Company A
xbe view customers list --name "<company-a-name>" --json

# Find all customer entities for Company B
xbe view customers list --name "<company-b-name>" --json
```

Note the customer IDs for each company - you'll need to include all of them to get complete data.

## Step 2: Retrieve Job Production Plans

Get plans for both companies over your comparison period (e.g., 3 months).

```bash
# Company A - get all plans for the period
xbe view job-production-plans list \
  --start-on-min <start-date> \
  --start-on-max <end-date> \
  --customer <customer-id-1>,<customer-id-2>,<customer-id-3> \
  --json > company_a_plans.json

# Company B - get all plans for the period
xbe view job-production-plans list \
  --start-on-min <start-date> \
  --start-on-max <end-date> \
  --customer <customer-id-1>,<customer-id-2> \
  --json > company_b_plans.json
```

## Step 3: Aggregate Metrics by Status

Different companies may use different status workflows (e.g., some keep plans as "approved", others move to "complete"). Analyze each status separately.

```bash
# Company A - aggregate by status
jq -r '
  group_by(.status) | 
  map({
    status: .[0].status,
    count: length,
    goal_tonnage: (map(.goal_tonnage // 0) | add),
    actual_tonnage: (map(.actual_tonnage // 0) | add),
    achievement_pct: ((map(.actual_tonnage // 0) | add) / (map(.goal_tonnage // 0) | add) * 100 | floor),
    avg_start_offset_minutes: ((map(.start_offset_seconds // 0) | add) / length / 60 | floor),
    truck_hours: (map(.truck_hours // 0) | add),
    truck_spend: (map(.truck_spend // 0) | add),
    actual_surplus_pct: ((map(.actual_surplus_percentage // 0) | add) / length | floor)
  }) | 
  map("\(.status): \(.count) plans, \(.achievement_pct)% achievement, \(.avg_start_offset_minutes) min offset, \(.actual_surplus_pct)% surplus") | 
  .[]' company_a_plans.json
```

## Step 4: Calculate Cost Efficiency

```bash
# Company A - cost per ton by status
jq -r '
  group_by(.status) | 
  map({
    status: .[0].status,
    total_tonnage: (map(.actual_tonnage // 0) | add),
    total_spend: (map(.truck_spend // 0) | add),
    cost_per_ton: ((map(.truck_spend // 0) | add) / (map(.actual_tonnage // 0) | add) | . * 100 | floor / 100)
  }) | 
  map("\(.status): $\(.cost_per_ton) per ton") | 
  .[]' company_a_plans.json
```

## Step 5: Analyze Monthly Trends

```bash
# Company A - monthly breakdown
jq -r '
  map(. + {month: (.start_on | split("-") | .[0:2] | join("-"))}) | 
  group_by(.month) | 
  map({
    month: .[0].month,
    count: length,
    achievement: ((map(.actual_tonnage // 0) | add) / (map(.goal_tonnage // 0) | add) * 100 | floor),
    avg_offset: ((map(.start_offset_seconds // 0) | add) / length / 60 | floor),
    surplus: ((map(.actual_surplus_percentage // 0) | add) / length | floor)
  }) | 
  sort_by(.month) | 
  map("\(.month): \(.count) plans, \(.achievement)% achievement, \(.avg_offset) min late, \(.surplus)% surplus") | 
  .[]' company_a_plans.json
```

## Step 6: Break Down by Customer Entity

Some divisions may perform very differently than others.

```bash
# Company A - by customer entity
jq -r '
  group_by(.customer.id) | 
  map({
    customer: .[0].customer.name,
    count: length,
    achievement: ((map(.actual_tonnage // 0) | add) / (map(.goal_tonnage // 0) | add) * 100 | floor),
    cost_per_ton: ((map(.truck_spend // 0) | add) / (map(.actual_tonnage // 0) | add) | . * 100 | floor / 100),
    avg_offset: ((map(.start_offset_seconds // 0) | add) / length / 60 | floor),
    surplus: ((map(.actual_surplus_percentage // 0) | add) / length | floor)
  }) | 
  map("\(.customer): \(.count) plans, \(.achievement)% achievement, $\(.cost_per_ton)/ton, \(.avg_offset) min late") | 
  .[]' company_a_plans.json
```

## Step 7: Analyze Cancellation Patterns

```bash
# Company A - cancellation rate
jq -r '
  (length) as $total | 
  (map(select(.status == "cancelled" or .status == "abandoned")) | length) as $cancelled | 
  "Total: \($total), Cancelled/Abandoned: \($cancelled), Rate: \(($cancelled * 100 / $total | floor))%"' company_a_plans.json
```

## Step 8: Create Comparison Report

Compile all metrics into a structured comparison report covering:
- Overall metrics (total plans, achievement rates, costs)
- Status workflow differences
- Monthly trends
- Entity-level breakdowns
- Cost efficiency comparisons
- On-time performance
- Surplus management
- Cancellation patterns

## Key Insights to Look For

1. **Status Workflow Maturity**: Does one company use more sophisticated status tracking (approved → complete) while the other keeps everything as "approved"?

2. **Execution Quality**: Compare achievement rates - consistent 80%+ is good, 50% suggests planning problems.

3. **Cost Efficiency**: Cost per ton varies significantly. Look for companies achieving <$10/ton on completed plans.

4. **Planning Discipline**: High cancellation rate (>40%) may indicate aggressive planning with good discipline to cancel unworkable plans rather than executing them poorly.

5. **Division Performance**: Some divisions may be star performers while others struggle. Compare Construction vs Materials vs Transport divisions.

6. **Trend Direction**: Are metrics improving or worsening over time? Recent trends matter more than overall averages.

7. **Surplus Management**: Completed plans should have 10-15% surplus. 40%+ suggests overcapacity and poor planning.

## Limitations

- Different status workflows make direct comparisons challenging
- Equipment movement plans may skew metrics (often 0 tons, not delivery plans)
- Some companies may be more diligent about tracking incidents and delays
- Different business models (construction vs materials supply) affect metrics
