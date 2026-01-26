---
title: Analyzing ML recommendation patterns across multiple shifts
when: When you need to understand how ML recommendations compare to actual assignments, analyze recommendation consistency across similar shifts, or evaluate whether dispatchers are following ML guidance
---

## Batch Recommendation Generation

When analyzing assignment patterns, generate recommendations for multiple related shifts to identify patterns:

```bash
# Generate recommendations for multiple shifts in a loop
for ljss_id in <shift-id-1> <shift-id-2> <shift-id-3>; do
  echo "=== Shift $ljss_id ==="
  
  # Create recommendation
  rec_id=$(xbe do lineup-job-schedule-shift-trucker-assignment-recommendations create \
    --lineup-job-schedule-shift $ljss_id \
    --json 2>/dev/null | jq -r '.id')
  
  # Show top 3 candidates
  xbe view lineup-job-schedule-shift-trucker-assignment-recommendations show $rec_id --json 2>/dev/null | \
    jq -r '.candidates[0:3] | .[] | "  Rank \(.rank): Trucker \(.trucker_id) - \(.probability*100 | round)%"'
  
  echo
done
```

## Identifying Recommendation Consistency

Similar shifts often produce identical recommendations, which helps identify patterns:

```bash
# Example output showing consistency:
# === Shift 3209873 ===
#   Rank 0: Trucker 336 - 37%
#   Rank 1: Trucker 338 - 14%
#   Rank 2: Trucker 257 - 6%
#
# === Shift 3209874 ===
#   Rank 0: Trucker 336 - 37%  # Same top recommendation
#   Rank 1: Trucker 338 - 14%  # Same probabilities
#   Rank 2: Trucker 257 - 6%
```

When all shifts show identical recommendations, it suggests:
- Similar shift characteristics (time, location, material, etc.)
- Consistent trucker availability patterns
- Predictable assignment factors

## Comparing Recommendations vs Actual Assignments

Cross-reference ML recommendations with actual assignments to understand dispatcher behavior:

```bash
# 1. Get actual assignment from lineup shift
xbe view lineup-job-schedule-shifts show <shift-id> --json | jq -r '."trucker-id"'
# Output: <actual-trucker-id>

# 2. Get ML recommendation
rec_id=$(xbe do lineup-job-schedule-shift-trucker-assignment-recommendations create \
  --lineup-job-schedule-shift <shift-id> --json | jq -r '.id')

xbe view lineup-job-schedule-shift-trucker-assignment-recommendations show $rec_id --json | \
  jq -r '.candidates[] | select(."trucker_id" == "<actual-trucker-id>") | "Actual assignment was rank \(.rank) with \(.probability*100 | round)% probability"'
```

## Analyzing Dispatcher Override Patterns

When actual assignments don't match top ML recommendations:

```bash
# Compare top recommendation vs actual
echo "ML Top Pick: $(jq -r '.candidates[0] | "Trucker \(.trucker_id) - \(.probability*100 | round)%"' rec.json)"
echo "Actual Assignment: Trucker <actual-id> was rank #<rank> (\<probability>%)"
```

**Common reasons for overrides:**
- Capacity constraints (top pick already at max shifts)
- Customer preferences or contracts
- Relationship management
- Fleet availability not captured in ML model
- Geographic/routing optimization

## Aggregating Recommendations for Job-Level Analysis

When a job has multiple shifts, aggregate recommendations to inform batch assignments:

```bash
# For all shifts in a job, count how often each trucker appears in top 3
for shift_id in $(xbe view lineup-job-schedule-shifts list --lineup <lineup-id> --json | \
  jq -r '.[] | select(."job-id" == "<job-id>") | .id'); do
  
  rec_id=$(xbe do lineup-job-schedule-shift-trucker-assignment-recommendations create \
    --lineup-job-schedule-shift $shift_id --json | jq -r '.id')
  
  xbe view lineup-job-schedule-shift-trucker-assignment-recommendations show $rec_id --json | \
    jq -r '.candidates[0:3] | .[]."trucker_id"'
done | sort | uniq -c | sort -rn

# Output shows which truckers appear most in top 3:
#  15 <trucker-a>  # Appears in top 3 for 15 shifts
#   8 <trucker-b>  # Appears in top 3 for 8 shifts
#   3 <trucker-c>  # Appears in top 3 for 3 shifts
```

This aggregation helps answer: "For this 10-shift job, which trucker(s) should handle it?"

## Validating ML Model Performance

Track how often dispatchers follow ML recommendations:

```bash
# For a lineup, calculate "ML followed" percentage
total=0
followed=0

for shift_id in $(get_all_shift_ids); do
  actual=$(get_actual_trucker $shift_id)
  ml_top=$(get_ml_top_pick $shift_id)
  
  total=$((total + 1))
  if [ "$actual" = "$ml_top" ]; then
    followed=$((followed + 1))
  fi
done

echo "ML Top Pick followed: $followed / $total ($((followed * 100 / total))%)"
```

## Important Notes

- Recommendations are generated for **lineup-job-schedule-shifts**, not job-schedule-shifts directly
- Each recommendation call creates a new recommendation resource
- Identical shift characteristics produce identical recommendations
- Low probabilities (< 10%) suggest unusual assignments or edge cases
- High agreement between ML and actual suggests good model calibration
