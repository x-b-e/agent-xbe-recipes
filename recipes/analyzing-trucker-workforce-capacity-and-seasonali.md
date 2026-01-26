---
title: Analyzing trucker workforce capacity and seasonality patterns
when: When you need to understand a trucker's operational capacity, fleet size, seasonal workforce scaling, or ability to flex capacity - especially for assessing replacement difficulty or concentration risk
---

# Analyzing trucker workforce capacity and seasonality patterns

## Problem
You need to understand a trucker's operational capacity, not just their volume. This includes:
- How many drivers/trucks they deploy
- How their workforce scales seasonally
- Whether they use full-time vs seasonal workers
- Their true fleet size and capacity constraints

This is especially important when assessing concentration risk or evaluating how difficult it would be to replace a trucker.

## Solution

Use lane summaries grouped by time periods with driver metrics to understand workforce patterns.

### Step 1: Get monthly unique driver counts

```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter trucker=<trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by month \
  --metrics unique_drivers \
  --sort month:asc \
  --json
```

This shows month-by-month workforce deployment.

### Step 2: Get weekly unique driver counts for granular seasonality

```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter trucker=<trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by week \
  --metrics unique_drivers \
  --sort week:asc \
  --json
```

Weekly data reveals:
- Peak capacity weeks
- Seasonal ramp-up/down patterns
- Week-to-week variability

### Step 3: Analyze driver utilization patterns

To understand core vs seasonal workforce:

```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter trucker=<trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by driver \
  --metrics days_active,trips_sum \
  --sort days_active:desc \
  --json
```

This reveals:
- How many drivers worked 20+ days (full-time core)
- How many worked 10-20 days (regular part-time)
- How many worked <10 days (seasonal/spot)

### Step 4: Combine with tonnage for capacity analysis

```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter trucker=<trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by month \
  --metrics unique_drivers,tons_sum,trips_sum \
  --sort month:asc \
  --json
```

Comparing drivers vs tonnage reveals:
- Tons per driver (productivity)
- Whether volume is driver-constrained
- Seasonal efficiency patterns

## Key insights to extract

### Seasonal scaling pattern
- **Winter low to peak ratio**: If 4 drivers in winter → 71 in peak, that's 18x variation
- **Ramp-up timeline**: How many months to scale from low to peak
- **Peak duration**: How many months sustained at peak capacity

### Workforce composition
From monthly data showing days worked:
- **Core full-time**: Drivers working 15+ days/month consistently
- **Seasonal regulars**: Drivers appearing May-Oct but not Nov-Apr
- **Spot/fill-in**: Drivers working sporadically

### Fleet size estimation
- **Active trucks**: Peak week unique drivers ≈ peak trucks deployed
- **Total fleet**: Peak active trucks × 1.4-1.8 (accounting for maintenance, rotation, reserves)
- Example: 71 peak drivers → estimated 100-130 total trucks

### Replacement difficulty indicators
- **High seasonal variation** (10x+ scaling): Hard to replace, requires sophisticated workforce management
- **Many unique drivers** (60+ at peak): Requires large driver pool or extensive owner-operator network
- **Consistent workforce**: If same drivers appear year-round, indicates stable employment model

## Example analysis output

After running these queries, you can produce insights like:

```
WORKFORCE CAPACITY ANALYSIS

Seasonal Pattern:
- Winter (Jan-Feb): 4-11 drivers
- Spring ramp (Mar-Apr): 14-40 drivers  
- Peak season (May-Oct): 45-71 drivers
- Fall decline (Nov): 45-23 drivers
- Seasonal variation: 18x (4 → 71)

Peak Capacity:
- Peak weeks: Week 37 (70 drivers), Week 40 (71 drivers)
- Average peak season: ~60 drivers/week
- Estimated fleet: 85-125 trucks total

Workforce Composition:
- Core full-time: ~40-50 drivers (present year-round)
- Seasonal flex: ~20-30 drivers (May-Oct only)
- Spot/fill-in: ~10-20 drivers (occasional)

Replacement Difficulty: HIGH
- Requires 18x seasonal scaling capability
- Needs 60-70 driver capacity during 6-month peak
- Sophisticated workforce management required
```

## Tips

- Use `--group-by week` for detailed seasonality, `--group-by month` for overview
- Compare `unique_drivers` vs `tons_sum` to assess productivity
- Look at `days_active` distribution to understand workforce stability
- Peak week drivers × 1.4-1.8 estimates total fleet size needed
- High seasonal variation (>10x) indicates sophisticated operations that are hard to replace
