---
title: Using ML-powered trucker assignment recommendations for lineup shifts
when: When you need to assign truckers to lineup shifts and want to use the system's ML-powered recommendation engine to identify the best candidates ranked by probability
---

# Using ML-powered trucker assignment recommendations for lineup shifts

The system provides an ML-powered recommendation engine that ranks potential truckers for a lineup shift by probability, making trucker assignment more data-driven.

## Basic workflow

### 1. Generate recommendations for a shift

```bash
xbe do lineup-job-schedule-shift-trucker-assignment-recommendations create \
  --lineup-job-schedule-shift <shift-id> \
  --json
```

Returns:
```json
{
  "id": "<recommendation-id>",
  "candidates_count": 76
}
```

### 2. View ranked candidates

```bash
xbe view lineup-job-schedule-shift-trucker-assignment-recommendations show <recommendation-id> \
  --fields candidates \
  --json
```

Returns candidates ranked by probability:
```json
{
  "candidates": [
    {
      "rank": 0,
      "trucker_id": <trucker-id>,
      "probability": 0.6668,
      "cumulative_probability": 0.6668,
      "score": -1.096
    },
    {
      "rank": 1,
      "trucker_id": <trucker-id>,
      "probability": 0.0214,
      "cumulative_probability": 0.6882,
      "score": -4.536
    }
  ]
}
```

### 3. View top N recommendations with jq

```bash
xbe view lineup-job-schedule-shift-trucker-assignment-recommendations show <recommendation-id> \
  --fields candidates \
  --json | jq '.candidates[0:5]'
```

### 4. Assign the top-ranked trucker

```bash
xbe do lineup-job-schedule-shifts update <shift-id> \
  --trucker <trucker-id> \
  --is-ready-to-dispatch true
```

## Understanding the output

- **rank**: 0-indexed ranking (0 = best match)
- **probability**: Individual probability for this trucker (0-1 scale)
- **cumulative_probability**: Sum of probabilities up to this rank
- **score**: Raw ML model score (lower/more negative = higher probability)
- **trucker_id**: The recommended trucker

## Interpreting probabilities

- **High confidence (>50%)**: Clear best choice, assign with confidence
- **Moderate confidence (20-50%)**: Strong candidate but consider context
- **Low confidence (<20%)**: Multiple viable options, may need human judgment
- **Close probabilities (35% vs 33%)**: May warrant review or business rule checks

## Bulk assignment pattern

```bash
for shift_id in <shift-id-1> <shift-id-2> <shift-id-3>; do
  # Generate recommendations
  rec_id=$(xbe do lineup-job-schedule-shift-trucker-assignment-recommendations create \
    --lineup-job-schedule-shift $shift_id --json | jq -r '.id')
  
  # Get top trucker
  top_trucker=$(xbe view lineup-job-schedule-shift-trucker-assignment-recommendations show $rec_id \
    --fields candidates --json | jq -r '.candidates[0].trucker_id')
  
  # Assign it
  xbe do lineup-job-schedule-shifts update $shift_id \
    --trucker $top_trucker \
    --is-ready-to-dispatch true
done
```

## Getting trucker names for recommendations

```bash
for trucker_id in <trucker-id-1> <trucker-id-2> <trucker-id-3>; do
  echo "Trucker $trucker_id: $(xbe view truckers show $trucker_id --json | jq -r '."company-name"')"
done
```

## When to use this vs manual assignment

**Use recommendations when:**
- Assigning many shifts in bulk
- You don't have specific business requirements for a particular trucker
- You want data-driven assignments based on historical patterns

**Override recommendations when:**
- Customer requires specific trucker
- Trucker availability constraints (capacity, equipment, driver unavailability)
- Business rules (e.g., "never assign X to Y on Mondays")
- Probabilities are very close and business context matters

## Re-assignment workflow

If the top-ranked trucker declines or is unavailable:

1. The recommendations are already generated
2. Pick the next-ranked trucker (rank 1, 2, etc.)
3. Update the assignment with the alternative trucker

No need to regenerate recommendations - the ranked list provides alternatives.
