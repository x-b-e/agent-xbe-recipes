---
title: Analyzing top lanes for a broker
when: When you need to identify the highest volume lanes (origin-destination pairs) for a specific broker over a date range
---

# Analyzing top lanes for a broker

To find the top lanes for a broker by tonnage:

1. First, find the broker's ID:
```bash
xbe view brokers list --company-name "<broker-name>"
```

2. Then create a lane summary grouped by origin and destination:
```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by origin,destination \
  --sort tons_sum:desc \
  --limit <number-of-lanes>
```

## Key points:
- Group by `origin,destination` to get distinct lanes
- Sort by `tons_sum:desc` to show highest volume lanes first
- Use `--limit` to control how many top lanes to return
- The result shows tonnage, cycle counts, and timing metrics for each lane

## Related recipes:
- For breaking down lanes by trucker: see "Analyzing lanes by trucker"
- For trucker-specific volume analysis: see "Analyzing highest volume lanes by trucker"
