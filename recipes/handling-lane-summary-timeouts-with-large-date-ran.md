---
title: Handling lane summary timeouts with large date ranges
when: When a lane summary query times out due to large date ranges or high data volume
---

# Handling lane summary timeouts with large date ranges

## Problem
When querying lane summaries with large date ranges (e.g., a full year) or complex groupings, the query may timeout with:
```
context deadline exceeded (Client.Timeout or context cancellation while reading body)
```

## Solutions

### Approach 1: Reduce the date range
Try a smaller time window (e.g., 6 months instead of 12):
```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by origin,destination,trucker \
  --sort tons_sum:desc
```

### Approach 2: Split into multiple periods and manually aggregate
For full-year analysis when a single query times out:

1. **First half of year:**
```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<year-start> \
  --filter date_max=<year-mid> \
  --group-by origin,destination,trucker \
  --sort tons_sum:desc
```

2. **Second half of year:**
```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<year-mid> \
  --filter date_max=<year-end> \
  --group-by origin,destination,trucker \
  --sort tons_sum:desc
```

3. **Manually combine results:**
   - For each unique lane-trucker combination, sum the tons from both periods
   - Add cycle counts and transaction counts from both periods
   - Present as H1 (first half) and H2 (second half) columns for transparency

### Approach 3: Simplify grouping
Remove one or more dimensions from `--group-by` to reduce query complexity:
```bash
# Instead of: --group-by origin,destination,trucker
# Try: --group-by origin,destination
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by origin,destination \
  --sort tons_sum:desc
```

## When to use each approach
- **Approach 1**: First attempt - try 6 months if 12 months times out
- **Approach 2**: When you need full-year data broken down by trucker and the query consistently times out
- **Approach 3**: When you can sacrifice detail (e.g., don't need trucker breakdown) to get results faster
