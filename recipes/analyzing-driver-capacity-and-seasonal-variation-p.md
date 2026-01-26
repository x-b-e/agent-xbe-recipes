---
title: Analyzing driver capacity and seasonal variation patterns
when: When you need to understand how a trucker's workforce scales throughout the year, identify peak vs off-season staffing levels, or estimate fleet size requirements based on active driver counts
---

# Analyzing Driver Capacity and Seasonal Variation Patterns

## Problem
You need to understand how a trucker's driver workforce scales throughout the year to:
- Identify seasonal hiring patterns
- Estimate core vs flex workforce
- Understand peak capacity requirements
- Validate fleet size estimates

## Solution

### Step 1: Get Weekly Unique Driver Counts

Create a script to iterate through weeks and count unique drivers:

```bash
cat > /tmp/count_drivers.sh << 'EOF'
#!/bin/bash
for week in {1..52}; do
  result=$(xbe summarize driver-day-summary create \
    --filter broker=<broker-id> \
    --filter trucker=<trucker-id> \
    --filter week=$week \
    --filter year=<year> \
    --group-by driver \
    --json 2>/dev/null)
  
  if [ -n "$result" ]; then
    count=$(echo "$result" | jq '[.rows[] | select(.driver_name != null)] | length')
    if [ "$count" -gt 0 ]; then
      echo "Week $week: $count"
    fi
  fi
done
EOF
chmod +x /tmp/count_drivers.sh
/tmp/count_drivers.sh
```

Key points:
- Use `driver-day-summary` grouped by driver to get unique drivers
- Filter by week to break down temporally
- Count non-null driver_name entries
- Suppress stderr with `2>/dev/null` to reduce noise

### Step 2: Analyze the Patterns

Look for:

**Seasonal Variation:**
- Winter low: Minimum driver counts (e.g., 4-11 drivers)
- Spring ramp-up: Growth period (e.g., 14-40 drivers)
- Peak season: Maximum capacity (e.g., 45-71 drivers)
- Fall decline: Reduction period
- Winter shutdown: End-of-year minimum

**Workforce Composition Estimates:**
- Core workforce: Lowest consistent count (active year-round)
- Seasonal workforce: Peak minus core (hired for busy season)
- Spot/flex capacity: Occasional additional drivers

**Fleet Size Validation:**
- Peak week driver count approximates active trucks needed
- Total fleet = peak active trucks × 1.5-2.0 (for maintenance/rotation)
- Example: 70 peak drivers suggests 105-140 total truck fleet

### Step 3: Cross-reference with Monthly Data

Use monthly driver-day summaries to validate and get additional detail:

```bash
xbe summarize driver-day-summary create \
  --filter broker=<broker-id> \
  --filter trucker=<trucker-id> \
  --filter month=<month> \
  --filter year=<year> \
  --group-by driver \
  --metrics days_worked_sum \
  --sort days_worked_sum:desc \
  --json | jq -r '.rows[] | [.driver_name, .days_worked_sum] | @tsv'
```

This shows:
- Total unique drivers for the month
- Distribution of full-time vs part-time workers
- Core workforce identification (drivers working most days)

## Example Insights

From a typical construction materials trucker:
- **18x seasonal variation** (4 winter drivers → 71 peak drivers)
- **Core workforce:** ~40-50 drivers (work year-round)
- **Seasonal additions:** ~20-30 drivers (May-October)
- **Spot capacity:** ~10-20 drivers (occasional fill-in)
- **Peak weeks:** Usually September-October before weather turns
- **No zero weeks:** Even slowest periods maintain minimal crew

## Common Patterns

**Construction materials hauling:**
- Strong seasonality (spring-fall peak)
- 3-5x capacity variation
- Core crew maintains winter operations

**Year-round operations:**
- Minimal variation (1.2-1.5x)
- Steady driver count
- Small flex capacity for peaks

**Hybrid seasonal:**
- Moderate variation (2-3x)
- Distinct but not extreme seasons
- Larger core workforce relative to peak

## Related Techniques

- For concentration risk analysis: See "Analyzing supplier concentration risk and strategic dependency"
- For overall trucker metrics: See "Analyzing tonnage by trucker for a broker"
- For comparing efficiency: See "Comparing trucker spend and efficiency between customers"
