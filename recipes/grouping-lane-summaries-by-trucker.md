---
title: Grouping lane summaries by trucker
when: When you need to analyze total volume, costs, or performance metrics broken down by each trucker serving a broker
---

To see aggregate metrics (volume, cycles, costs) broken down by trucker:

```bash
xbe summarize lane-summary create \
  --group-by trucker \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --json
```

This shows you which truckers are handling what volume for a broker, useful for:
- Market share analysis by trucker
- Comparing costs across truckers
- Identifying dependencies on specific truckers

You can add `--metric effective_cost_per_hour_median` or other metrics to compare trucker economics.
