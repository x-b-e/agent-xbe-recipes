---
title: Analyzing supplier concentration risk and strategic dependency
when: When you need to assess how dependent a broker is on a specific trucker (or vice versa), quantify concentration risk, evaluate alternative capacity, estimate fleet requirements for replacement scenarios, and develop risk mitigation strategies
---

# Analyzing supplier concentration risk and strategic dependency

When analyzing the strategic dependency between a broker and their primary trucker, you need to quantify the relationship from multiple angles: volume concentration, financial dependency, operational capacity, and replacement feasibility.

## Step 1: Identify the broker and trucker

```bash
# Find the broker ID
xbe view brokers list --company-name "<broker-name>" --json

# Find the trucker ID
xbe view truckers list --name "<trucker-name>" --json
```

## Step 2: Analyze volume concentration

Get the broker's total tonnage broken down by trucker to see concentration:

```bash
# Recent period (avoid timeouts with large date ranges)
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by trucker \
  --metrics tons_sum \
  --sort tons_sum:desc \
  --limit 50 \
  --json
```

If you get timeouts with large date ranges, break into smaller periods:

```bash
# Current year
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<current-year>-01-01 \
  --filter date_max=<current-date> \
  --group-by trucker \
  --metrics tons_sum \
  --sort tons_sum:desc \
  --limit 50 \
  --json

# Previous year for comparison
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<previous-year>-01-01 \
  --filter date_max=<previous-year>-12-31 \
  --group-by trucker \
  --metrics tons_sum \
  --sort tons_sum:desc \
  --limit 50 \
  --json
```

## Step 3: Calculate concentration metrics

Use jq to calculate what percentage of volume comes from the primary trucker:

```bash
# From the JSON output above
cat <output-file> | jq '
  .values as $data |
  ($data[0][1] | tonumber) as $top_trucker |
  ($data | map(.[1] | tonumber) | add) as $total |
  {
    top_trucker_name: $data[0][0],
    top_trucker_tons: $top_trucker,
    total_tons: $total,
    concentration_pct: (($top_trucker / $total) * 100 | round)
  }'
```

## Step 4: Analyze the reverse dependency

Get the trucker's volume broken down by broker to see how dependent they are on this broker:

```bash
xbe summarize lane-summary create \
  --filter trucker=<trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by broker \
  --metrics tons_sum \
  --sort tons_sum:desc \
  --json
```

## Step 5: Analyze monthly patterns and trends

Look at the relationship over time to identify trends:

```bash
# Monthly breakdown for the primary relationship
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter trucker=<trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by month \
  --metrics tons_sum,trips_sum,total_cost_sum \
  --json
```

## Step 6: Estimate fleet size requirements for replacement scenarios

To understand what it would take to replace the primary trucker, calculate their fleet size:

### Method 1: Annual volume-based calculation

```bash
# Industry standard: ~13,500 tons per truck per year for aggregate hauling
# Annual tons ÷ 13,500 = approximate truck count
```

### Method 2: Peak month analysis

Identify the peak month and calculate daily requirements:

```bash
# Get monthly data to find peak
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter trucker=<trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by month \
  --metrics tons_sum,trips_sum \
  --json

# Calculate from peak month:
# <peak-tons> ÷ 25 tons/load = <trips-in-peak-month>
# <trips-in-peak-month> ÷ 22 working days = <trips-per-day>
# <trips-per-day> ÷ 3 trips/truck/day = <trucks-needed-in-peak>
```

### Method 3: Driver and shift analysis

Analyze actual driver data to validate fleet size estimates:

```bash
# Get driver activity for peak month
xbe summarize shift-summary create \
  --filter broker=<broker-id> \
  --filter trucker=<trucker-id> \
  --filter date_min=<peak-month-start> \
  --filter date_max=<peak-month-end> \
  --group-by driver \
  --metrics driver_day_count,shift_count_sum,duration_hours_sum \
  --json
```

From shift data:
- Count drivers working 15+ days (full-time core fleet)
- Count drivers working 5-14 days (part-time/relief)
- Count drivers working 1-4 days (spot/seasonal)
- Look for null driver entries (unattributed shifts suggest owner-operators)

### Typical fleet composition for aggregate haulers:

```
Core Fleet: 50-65% of capacity
- Company-owned power units
- Full-time drivers
- Predictable, controllable capacity

Owner-Operator Partners: 30-40% of capacity  
- Dedicated or semi-dedicated relationships
- Lower capital requirements for the trucker
- Need strong relationships to ensure availability

Seasonal/Spot Capacity: 10-20% of capacity
- Peak season augmentation (April-November)
- Fill-in during maintenance/breakdowns
- Higher per-trip cost, no fixed cost
```

### Equipment requirements:

```
End dump trailers (standard aggregate hauling)
- 22-25 ton capacity per load
- Day cabs for local/regional work  
- Mix of newer fleet (company) + older (owner-operators)
```

## Step 7: Evaluate alternative capacity

Analyze the capacity of other truckers in the network:

```bash
# Get all truckers sorted by volume
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by trucker \
  --metrics tons_sum \
  --sort tons_sum:desc \
  --limit 50 \
  --json
```

Calculate how many of the next-largest truckers would be needed:

```bash
cat <output-file> | jq '
  .values as $data |
  ($data[0][1] | tonumber) as $primary_tons |
  ($data[1:] | map(.[1] | tonumber)) as $other_tons |
  {
    primary_tons: $primary_tons,
    next_5_combined: ($other_tons[0:5] | add),
    next_10_combined: ($other_tons[0:10] | add),
    truckers_needed_to_match: ($other_tons | [foreach .[] as $t (0; . + $t; . >= $primary_tons)] | index(true) + 1)
  }'
```

## Step 8: Calculate financial metrics

```bash
# Revenue from primary trucker relationship
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter trucker=<trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --metrics total_cost_sum,tons_sum \
  --json

# Cost efficiency comparison
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by trucker \
  --metrics tons_sum,total_cost_sum,duration_hours_sum \
  --json
```

Calculate cost per ton and effective hourly rate:

```bash
cat <output-file> | jq '
  .values | map({
    trucker: .[0],
    tons: (.[1] | tonumber),
    cost: (.[2] | tonumber),
    hours: (.[3] | tonumber),
    cost_per_ton: ((.[2] | tonumber) / (.[1] | tonumber)),
    cost_per_hour: ((.[2] | tonumber) / (.[3] | tonumber))
  }) | sort_by(.cost_per_ton)'
```

## Key Questions to Answer

1. **Concentration Risk**: What % of broker's volume comes from this trucker?
2. **Mutual Dependency**: What % of trucker's volume comes from this broker?
3. **Replacement Capacity**: How many trucks would be needed to replace them? What fleet size?
4. **Alternative Options**: How many other truckers would it take to match capacity?
5. **Financial Impact**: What is the revenue/cost at risk?
6. **Cost Position**: Is the primary trucker cost-competitive or premium-priced?
7. **Seasonal Factors**: Does dependency vary by season? Peak vs off-peak needs?
8. **Fleet Composition**: What mix of company trucks vs owner-operators?
9. **Growth Trajectory**: Is the relationship growing, stable, or declining?
10. **Time to Replace**: How long would it take to build alternative capacity?

## Mitigation Strategies to Research

1. **Diversification**: Systematically grow 2-3 alternative truckers to 30-40% of primary's capacity each
2. **Market Development**: Recruit large truckers from adjacent markets
3. **Contractual Protection**: Long-term contracts, minimum volume commitments, exit penalties
4. **Internal Capacity**: Build or acquire internal trucking operation (major capital decision)
5. **Owner-Operator Network**: Develop direct relationships with owner-operators
6. **Seasonal Planning**: Different capacity strategies for peak vs off-peak seasons
7. **Performance Monitoring**: Track service levels, cost trends, capacity utilization
8. **Relationship Management**: Executive relationships, joint planning, transparency

## Example Calculation

Given:
- Primary trucker handles 1,716,279 tons annually
- Peak month: 255,219 tons in September
- 82 named drivers + 876 unattributed driver-days in peak month
- 1,252 shifts in peak month

Fleet size estimates:
- **Annual average method**: 1,716,279 ÷ 13,500 = ~127 trucks
- **Peak daily method**: 255,219 tons ÷ 22 days = 11,601 tons/day; 11,601 ÷ 25 tons/load = 464 trips/day; 464 ÷ 3 trips/truck = ~155 trucks needed in peak
- **Conclusion**: Fleet likely sized at 120-140 trucks (100-120 base + 20-40 flex capacity)

Replacement challenge:
- Next largest trucker handles only 143K tons (equivalent of ~11 trucks)
- Would need to combine 10-12 other truckers to match capacity
- Building a 120-140 truck operation requires $25-50M+ in assets and takes years
