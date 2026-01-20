---
title: Including effective cost per hour in lane summaries
when: When you need to see profitability metrics (effective rate/hour) alongside tonnage and cycle data in lane summaries
---

To include the effective cost per hour metric in a lane summary, add `effective_cost_per_hour_median` to the `--metrics` flag.

## Example

```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter trucker=<trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by origin,destination \
  --metrics tons_sum,cycle_count,material_transaction_count,effective_cost_per_hour_median \
  --sort tons_sum:desc
```

## Key Points

- The metric name is `effective_cost_per_hour_median` (not just `effective_cost_per_hour`)
- This can be combined with other metrics like `tons_sum`, `cycle_count`, `material_transaction_count`
- The value will be formatted as currency (e.g., `$125.29`)
- Use this to compare profitability across different lanes or truckers
