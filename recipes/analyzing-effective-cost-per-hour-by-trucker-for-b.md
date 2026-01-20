---
title: Analyzing effective cost per hour by trucker for broker lanes
when: When you need to compare trucking costs (effective rate per hour) across different truckers and lanes for a broker to identify expensive vs efficient routes
---

# Analyzing effective cost per hour by trucker for broker lanes

## Use Case
When you need to compare the effective cost per hour across different truckers operating lanes for a broker, to identify which truckers are most/least expensive on which routes.

## Prerequisites
- Know the broker ID (use `xbe view brokers list --company-name "<company-name>"` if needed)

## Command Pattern

```bash
xbe summarize lane-summary create \
  --group-by origin,destination,trucker \
  --filter broker=<broker-id> \
  --filter year=<year> \
  --metrics effective_cost_per_hour_median,cycle_count,tons_sum,material_transaction_count \
  --min-transactions <min-count> \
  --limit <limit>
```

## Key Flags

- `--group-by origin,destination,trucker`: Groups data by lane (origin-destination) AND trucker to see cost breakdowns
- `--metrics effective_cost_per_hour_median`: Include the cost efficiency metric
- `--min-transactions <min-count>`: Filter to only high-volume lanes (e.g., 25 or 50+)
- Additional useful metrics: `cycle_count`, `tons_sum`, `material_transaction_count` for context

## Example Analysis Pattern

1. **Get high-volume lane costs by trucker:**
```bash
xbe summarize lane-summary create \
  --group-by origin,destination,trucker \
  --filter broker=<broker-id> \
  --filter year=<year> \
  --metrics effective_cost_per_hour_median,cycle_count,tons_sum,material_transaction_count \
  --min-transactions 50 \
  --limit 100
```

2. **Sort by cost (descending) to find most expensive lanes:**
Note: The CLI may support `--sort effective_cost_per_hour_median:desc` but sorting can also be done post-processing

3. **Analyze patterns:**
- Which truckers consistently have higher/lower costs?
- Which lanes (origin-destination pairs) are expensive across all truckers?
- Which trucker-lane combinations are most efficient?

## Common Insights

- **Trucker comparison**: Compare internal fleet vs external truckers on same lanes
- **Lane characteristics**: Some lanes may be inherently expensive (longer distance, difficult access)
- **Volume vs efficiency**: High-volume lanes may not be the most cost-efficient
- **Optimization opportunities**: Identify lanes where switching truckers or negotiating rates could reduce costs

## Related Recipes

- [Finding broker ID by company name](finding-broker-id-by-company-name.md)
- [Analyzing highest volume lanes by trucker](analyzing-highest-volume-lanes-by-trucker.md)
- [Including effective cost per hour in lane summaries](including-effective-cost-per-hour-in-lane-summaries.md)
