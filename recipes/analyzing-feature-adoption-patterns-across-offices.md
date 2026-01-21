---
title: Analyzing feature adoption patterns across offices over time
when: When you need to analyze how different offices/branches adopted a feature over time, calculate adoption velocity metrics (time to reach thresholds), identify patterns (rapid vs gradual adoption), and generate insights about rollout success
---

# Analyzing Feature Adoption Patterns Across Offices Over Time

This recipe shows how to analyze feature adoption across multiple offices over time by fetching weekly snapshots, calculating adoption percentages, and identifying patterns.

## Use Case

Analyzing managed transport adoption across Quantix offices to understand:
- How quickly offices adopted managed transport (time to reach 50%, 80% managed)
- Whether adoption follows a "flip the switch" or "gradual ramp" pattern
- Whether later adopters ramp faster than early adopters (learning transfer)
- Which offices are lagging and may need intervention

## Step 1: Identify the date range and cadence

Determine the analysis period and sampling frequency:
- Start date: When the feature became available
- End date: Current week
- Cadence: Weekly snapshots (adjust based on data volume and adoption speed)

## Step 2: Fetch weekly snapshots for each office

Use a loop to fetch data for each week, filtering by office:

```bash
# Get list of offices first
xbe view brokers show <broker-id> --json | jq -r '.offices[] | "\(.id),\(.name)"'

# For each week and each office, fetch transport orders
for week in <week-list>; do
  START_DATE=$(date -j -f "%Y-W%U-%u" "${week}-1" +"%Y-%m-%d")
  END_DATE=$(date -j -f "%Y-W%U-%u" "${week}-7" +"%Y-%m-%d")
  
  for office_id in <office-ids>; do
    # Fetch total orders
    TOTAL=$(xbe view transport-orders list \
      --broker <broker-id> \
      --project-office ${office_id} \
      --start-on ${START_DATE} \
      --end-on ${END_DATE} \
      --json | jq 'length')
    
    # Fetch managed orders
    MANAGED=$(xbe view transport-orders list \
      --broker <broker-id> \
      --project-office ${office_id} \
      --start-on ${START_DATE} \
      --end-on ${END_DATE} \
      --is-managed true \
      --json | jq 'length')
    
    # Calculate percentage
    if [ "$TOTAL" -gt 0 ]; then
      PCT=$(echo "scale=2; ($MANAGED * 100) / $TOTAL" | bc)
    else
      PCT=0
    fi
    
    echo "${week},${office_id},${TOTAL},${MANAGED},${PCT}"
  done
done > adoption_data.csv
```

## Step 3: Calculate adoption metrics

Process the weekly data to identify:
- Adoption start week (first week with >5% managed)
- Time to 50% managed
- Time to 80% managed
- Current adoption percentage

```bash
# Example: Calculate time to 80% for each office
cat adoption_data.csv | \
  awk -F, '
    $5 > 5 && !start[$2] { start[$2] = $1 }  # First week >5%
    $5 >= 80 && !hit80[$2] { hit80[$2] = $1 }  # First week >=80%
    END {
      for (office in start) {
        if (hit80[office]) {
          # Calculate week difference
          split(start[office], s, "-W")
          split(hit80[office], h, "-W")
          weeks = h[2] - s[2]
          print office, weeks
        }
      }
    }'
```

## Step 4: Identify patterns

Analyze the data to categorize offices:

**Rapid adopters:** Reached 80% in ≤2 weeks
**Moderate adopters:** Reached 80% in 3-4 weeks
**Gradual adopters:** Reached 80% in 5+ weeks
**Stalled adopters:** Started (>5%) but haven't reached 80% after many weeks

```bash
# Categorize offices by adoption speed
cat time_to_80.csv | \
  awk '{
    if ($2 <= 2) print "Rapid:", $1, $2
    else if ($2 <= 4) print "Moderate:", $1, $2
    else print "Gradual:", $1, $2
  }'
```

## Step 5: Analyze early vs late adopters

Compare adoption speed by cohort to identify learning transfer:

```bash
# Group by start week and calculate average time to 80%
cat adoption_data.csv | \
  awk -F, '
    # Track start week and time to 80% for each office
    ...
  ' | \
  sort -t, -k1,1n  # Sort by start week
```

## Step 6: Generate insights and recommendations

Based on the analysis:

1. **Calculate statistics:**
   - Median/average time to 50% and 80%
   - Percentage following "rapid flip" vs "gradual ramp"
   - Trend in adoption speed over time (early vs late adopters)

2. **Identify laggards:**
   - Offices with >X weeks in progress but <80%
   - Offices that started but show no progress

3. **Project remaining ramp:**
   - Based on typical time-to-threshold, estimate when overall target will be reached
   - Create scenarios (aggressive/moderate/conservative)

4. **Actionable recommendations:**
   - Focus on "flip readiness" if pattern is binary
   - Study fast adopters to create playbook
   - Diagnose blockers for stalled offices

## Key Learnings

- **Adoption patterns are often binary, not linear:** Offices tend to "flip" from low to high adoption quickly once ready, rather than gradually ramping
- **Later adopters benefit from knowledge transfer:** Time-to-threshold typically decreases for later adopters
- **Stalled offices need diagnosis:** If an office starts but doesn't progress, there's likely a specific blocker to address
- **Use weekly snapshots for sufficient granularity** while avoiding excessive API calls

## Variations

- **Different thresholds:** Adjust 50%/80% targets based on your success criteria
- **Different cadences:** Daily for fast-moving rollouts, monthly for slow adoption
- **Different filters:** Analyze by region, customer segment, or other dimensions
- **Different features:** Apply same methodology to any feature with measurable adoption (confirmations, planning, etc.)
