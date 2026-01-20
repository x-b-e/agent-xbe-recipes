---
title: Analyzing highest volume lanes by trucker
when: When you need to identify top lanes broken down by trucker to see which truckers handle the highest volumes on which routes for a broker
---

# Analyzing highest volume lanes by trucker

## Purpose
Identify the highest volume lanes (origin-destination pairs) broken down by trucker to understand which truckers are handling the most tonnage on which routes.

## Command Pattern

```bash
xbe summarize lane-summary create \
  --group-by origin,destination,trucker \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --sort tons_sum:desc
```

## Key Parameters

- `--group-by origin,destination,trucker`: Groups results by origin site, destination site, and trucker to show each unique lane-trucker combination
- `--sort tons_sum:desc`: Sorts by total tons in descending order to show highest volume lanes first
- Date filters: Use `date_min` and `date_max` for filtering by date range

## Understanding the Output

The output shows:
- `origin_name`: The pickup/origin site
- `destination_name`: The delivery/destination site  
- `trucker_name`: The trucking company handling the lane
- `cycle_count`: Number of completed cycles
- `material_transaction_count`: Number of individual transactions
- `tons_sum`: Total tons hauled (the key metric for volume)

## Analysis Tips

1. **Identify top performing truckers**: Look for truckers appearing frequently in the top results
2. **Spot lane concentration**: See which origin-destination pairs have the highest volumes
3. **Compare trucker performance**: Multiple truckers on the same lane can be compared side-by-side
4. **Understand fleet utilization**: See which truckers dominate which geographic routes

## Related Recipes

- For focusing on a specific trucker's lanes, see "Analyzing lanes by trucker"
- For getting broker totals without breakdowns, see "Getting aggregate totals without grouping"
