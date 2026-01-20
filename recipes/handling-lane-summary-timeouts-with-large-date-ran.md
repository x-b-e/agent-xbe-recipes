---
title: Handling lane summary timeouts with large date ranges
when: When a lane summary query times out due to large date ranges or high data volume
---

# Handling lane summary timeouts with large date ranges

## Problem
Lane summary queries can timeout when:
- Date ranges are too large (e.g., full year)
- The broker has high transaction volume
- Multiple group-by dimensions create large result sets

## Solution
Reduce the date range incrementally until the query succeeds:

```bash
# If this times out:
xbe summarize lane-summary create --filter broker=<broker-id> \
  --filter date_min=<start-date> --filter date_max=<end-date> \
  --group-by origin,destination,trucker --sort tons_sum:desc

# Try reducing to 6 months:
xbe summarize lane-summary create --filter broker=<broker-id> \
  --filter date_min=<6-months-ago-date> --filter date_max=<end-date> \
  --group-by origin,destination,trucker --sort tons_sum:desc

# Or try 3 months if still timing out:
xbe summarize lane-summary create --filter broker=<broker-id> \
  --filter date_min=<3-months-ago-date> --filter date_max=<end-date> \
  --group-by origin,destination,trucker --sort tons_sum:desc
```

## Alternative approaches
- Remove group-by dimensions to reduce result size
- Add more specific filters (e.g., filter by specific origins or destinations)
- Break the query into multiple smaller time periods and combine results manually

## When to use smaller ranges
- High-volume brokers (thousands of transactions per month)
- Queries grouping by multiple dimensions (origin, destination, trucker)
- When you see "context deadline exceeded" errors
