---
title: Analyzing adoption velocity across offices over time
when: When you need to analyze how quickly different offices/entities adopted a feature (like managed transport) over time, identify patterns in adoption speed, calculate time-to-milestone metrics, or project future adoption timelines
---

# Analyzing adoption velocity across offices over time

## Use Case
Analyze how quickly different offices adopted managed transport (or any similar feature) to:
- Identify rapid vs gradual adopters
- Calculate time-to-milestone statistics (e.g., weeks to reach 50% or 80% adoption)
- Compare early vs late adopters
- Project future adoption timelines

## Step-by-Step Process

### 1. Get weekly time-series data for all offices

```bash
# Fetch managed transport data for each week, output as JSON
for week in <week-start> <week-2> <week-3> <week-end>; do
  xbe view transport-orders list \
    --broker <broker-id> \
    --start-on "$week" \
    --end-on "$week" \
    --json >> /tmp/weekly_data.jsonl
done
```

### 2. Extract adoption percentages by office and week

Use `jq` to process the JSON data:

```bash
jq -s '
  # Group all orders by office and week
  map(select(.transportOrders != null) | 
    .transportOrders[] | 
    {office: .projectOffice, week: .pickupAt, isManaged: .isManaged}
  ) |
  group_by(.office) |
  map({
    office: .[0].office,
    weeks: (group_by(.week) | map({
      week: .[0].week,
      total: length,
      managed: ([.[] | select(.isManaged == true)] | length),
      pct: (([.[] | select(.isManaged == true)] | length) / length * 100)
    }))
  })
' /tmp/weekly_data.jsonl > /tmp/adoption_by_office.json
```

### 3. Identify adoption milestones

Find when each office first crossed key thresholds:

```bash
jq -r '
  .[] |
  . as $office |
  {
    office: .office,
    firstStart: ([.weeks[] | select(.pct > 5)] | first | .week // null),
    first50: ([.weeks[] | select(.pct >= 50)] | first | .week // null),
    first80: ([.weeks[] | select(.pct >= 80)] | first | .week // null),
    current: (.weeks | last | .pct)
  } |
  [.office, .firstStart, .first50, .first80, .current] |
  @tsv
' /tmp/adoption_by_office.json > /tmp/milestones.tsv
```

### 4. Calculate time-to-milestone metrics

```bash
# Calculate weeks from start to 80% adoption
jq -r '
  .[] |
  select(.firstStart != null and .first80 != null) |
  {
    office: .office,
    weeksTo80: (((.first80 | fromdateiso8601) - (.firstStart | fromdateiso8601)) / 604800 | floor)
  } |
  [.office, .weeksTo80] | @tsv
' /tmp/adoption_by_office.json | 
sort -t$'\t' -k2 -n > /tmp/velocity_ranking.tsv
```

### 5. Identify adoption patterns

Categorize offices by adoption speed:

```bash
# Rapid adopters (≤2 weeks to 80%)
jq -r '
  .[] |
  select(.weeksTo80 != null and .weeksTo80 <= 2) |
  .office
' /tmp/velocity_ranking.json

# Gradual adopters (5+ weeks to 80%)
jq -r '
  .[] |
  select(.weeksTo80 != null and .weeksTo80 >= 5) |
  .office
' /tmp/velocity_ranking.json
```

### 6. Analyze early vs late adopter trends

```bash
# Compare adoption speed by cohort
jq -s '
  group_by(.startWeek) |
  map({
    cohort: .[0].startWeek,
    avgWeeksTo80: ([.[] | .weeksTo80] | add / length),
    offices: length
  })
' /tmp/milestones.json
```

### 7. Calculate statistics

```bash
# Get median, average, min, max time to 80%
jq -s '
  [.[] | .weeksTo80] | sort |
  {
    median: (.[length/2 | floor]),
    average: (add / length),
    min: .[0],
    max: .[-1]
  }
' /tmp/velocity_ranking.json
```

## Key Insights to Extract

1. **Adoption pattern**: Do offices ramp gradually or "flip the switch" quickly?
2. **Learning curve effect**: Do later adopters ramp faster than early adopters?
3. **Time-to-milestone distribution**: What's typical vs outlier performance?
4. **Current state**: How many offices are at each stage of adoption?
5. **Projection**: Based on historical velocity, when will remaining offices reach targets?

## Creating a Timeline Projection

For remaining offices below target:

```bash
# Identify offices still ramping
jq -r '
  .[] |
  select(.current < 80) |
  {
    office: .office,
    currentPct: .current,
    weeksSinceStart: ((now - (.firstStart | fromdateiso8601)) / 604800 | floor),
    projectedWeeksRemaining: (<median-weeks-to-80> - ((now - (.firstStart | fromdateiso8601)) / 604800 | floor))
  }
' /tmp/adoption_by_office.json
```

## Tips

- **Start week identification**: Consider offices "started" when they first exceed 5% to filter out noise
- **Date calculations**: Use `fromdateiso8601` and Unix timestamps for week calculations
- **Cohort analysis**: Group adopters by start month/quarter to identify learning transfer effects
- **Outlier investigation**: Offices significantly faster or slower than median warrant investigation
