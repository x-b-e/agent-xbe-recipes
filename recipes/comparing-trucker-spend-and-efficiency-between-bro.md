---
title: Comparing trucker spend and efficiency between brokers
when: When you need to compare trucking costs and operational efficiency between two brokers, including per-ton costs, hourly rates, trucker rankings, and identifying cost optimization opportunities
---

# Comparing trucker spend and efficiency between brokers

## Overview
This recipe shows how to analyze and compare trucker spending patterns between two brokers to identify cost differences, efficiency gaps, and optimization opportunities.

## Steps

### 1. Get material transaction lane summaries for each broker

For each broker, retrieve lane summaries grouped by trucker with cost metrics:

```bash
xbe view material-transactions lane-summary \
  --broker <broker-id> \
  --hauled-on-min <start-date> \
  --hauled-on-max <end-date> \
  --group-by trucker_id \
  --include-effective-cost-per-hour \
  --json > <broker-name>_lanes.json
```

### 2. Transform and aggregate data by trucker

Use jq to aggregate metrics by trucker:

```bash
cat <broker-name>_lanes.json | jq -r '
  group_by(.trucker_id) | 
  map({
    trucker_id: .[0].trucker_id,
    trucker_name: .[0].trucker_name,
    shifts: map(.loads) | add,
    tons: map(.quantity_tons) | add,
    total_spend: map(.total_trucker_cost) | add,
    total_hours: map(.total_trucker_hours) | add
  }) |
  map(. + {
    cost_per_ton: (if .tons > 0 then .total_spend / .tons else 0 end),
    cost_per_hour: (if .total_hours > 0 then .total_spend / .total_hours else 0 end)
  }) |
  sort_by(-.total_spend)
' > <broker-name>_trucker_summary.json
```

### 3. Calculate aggregate metrics

Calculate overall averages:

```bash
cat <broker-name>_trucker_summary.json | jq '
  {
    total_shifts: map(.shifts) | add,
    total_tons: map(.tons) | add,
    total_spend: map(.total_spend) | add,
    avg_cost_per_ton: (map(.total_spend) | add) / (map(.tons) | add),
    avg_cost_per_hour: (map(.total_spend) | add) / (map(.total_hours) | add)
  }
'
```

### 4. Identify unmanaged truckers

Find truckers with shifts but no cost data:

```bash
cat <broker-name>_lanes.json | jq -r '
  map(select(.total_trucker_cost == 0 or .total_trucker_cost == null)) |
  group_by(.trucker_id) |
  map({
    trucker_name: .[0].trucker_name,
    shifts: map(.loads) | add,
    tons: map(.quantity_tons) | add
  })
'
```

### 5. Compare key metrics

Key comparisons to make:

**Volume Metrics:**
- Total shifts and tonnage
- Top trucker volumes (identify if one broker has a dominant high-volume trucker)

**Cost Metrics:**
- Average cost per ton (overall efficiency)
- Average cost per hour (rate comparison)
- Cost per ton by trucker (identify expensive vs efficient truckers)

**Efficiency Indicators:**
- Hourly rate vs per-ton cost relationship (higher $/hour with lower $/ton indicates better productivity)
- Unmanaged trucker percentage (financial visibility)
- Trucker concentration (dependency on specific truckers)

**Cost Anomalies:**
- Identify truckers with costs significantly above/below average
- Look for specialized work vs inefficiency

### 6. Key insights to extract

**Competitive Advantages:**
- Dominant trucker relationships with exceptional rates
- Better productivity metrics (lower $/ton despite higher $/hour)

**Problem Areas:**
- Truckers charging significantly above market rates
- Large volumes with unmanaged/untracked costs
- High-cost truckers with significant spend

**Optimization Opportunities:**
- Underutilized low-cost truckers
- High-volume truckers with room for rate negotiation
- Productivity gaps ($/hour vs $/ton efficiency)

## Example Analysis Pattern

```bash
# 1. Get data for both brokers
xbe view material-transactions lane-summary \
  --broker <broker-1-id> \
  --hauled-on-min <start-date> \
  --hauled-on-max <end-date> \
  --group-by trucker_id \
  --include-effective-cost-per-hour \
  --json > broker1_lanes.json

xbe view material-transactions lane-summary \
  --broker <broker-2-id> \
  --hauled-on-min <start-date> \
  --hauled-on-max <end-date> \
  --group-by trucker_id \
  --include-effective-cost-per-hour \
  --json > broker2_lanes.json

# 2. Create trucker summaries
for file in broker1_lanes.json broker2_lanes.json; do
  cat $file | jq -r '
    group_by(.trucker_id) | 
    map({
      trucker_id: .[0].trucker_id,
      trucker_name: .[0].trucker_name,
      shifts: map(.loads) | add,
      tons: map(.quantity_tons) | add,
      total_spend: map(.total_trucker_cost) | add,
      total_hours: map(.total_trucker_hours) | add
    }) |
    map(. + {
      cost_per_ton: (if .tons > 0 then .total_spend / .tons else 0 end),
      cost_per_hour: (if .total_hours > 0 then .total_spend / .total_hours else 0 end)
    }) |
    sort_by(-.total_spend)
  ' > ${file%.json}_summary.json
done

# 3. Compare top truckers and identify patterns
echo "Broker 1 Top 5 by Spend:"
cat broker1_lanes_summary.json | jq -r '.[:5] | .[] | "\(.trucker_name): $\(.total_spend | round) (\(.tons | round) tons at $\(.cost_per_ton | round) /ton)"'

echo "\nBroker 2 Top 5 by Spend:"
cat broker2_lanes_summary.json | jq -r '.[:5] | .[] | "\(.trucker_name): $\(.total_spend | round) (\(.tons | round) tons at $\(.cost_per_ton | round) /ton)"'
```

## Related Recipes
- analyzing-tonnage-by-trucker-for-a-broker.md - Simpler tonnage-only analysis
- including-effective-cost-per-hour-in-lane-summaries.md - Understanding cost metrics
- creating-supplier-leverage-analysis-report-cards.md - Analyzing power dynamics with truckers
