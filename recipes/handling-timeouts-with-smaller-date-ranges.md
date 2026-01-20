---
title: Handling timeouts with smaller date ranges
when: When lane summary queries timeout due to large date ranges or high-volume brokers
---

# Handling timeouts with smaller date ranges

When querying lane summaries for large date ranges or high-volume brokers, you may encounter timeout errors. The solution is to split the query into smaller time periods and combine the results.

## The Problem

```bash
# This may timeout for high-volume brokers
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by origin,destination,trucker \
  --sort tons_sum:desc
```

## The Solution

Split the date range into smaller periods (e.g., 6 months or quarters):

```bash
# First half of the year
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<first-half-start> \
  --filter date_max=<first-half-end> \
  --group-by origin,destination,trucker \
  --sort tons_sum:desc

# Second half of the year
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<second-half-start> \
  --filter date_max=<second-half-end> \
  --group-by origin,destination,trucker \
  --sort tons_sum:desc
```

## Combining Results

Once you have the results from each period, you can:

1. Manually aggregate the top lanes by adding tonnage from both periods
2. Use the data to identify consistent high-volume lanes across periods
3. Compare performance between periods (H1 vs H2)

## Tips

- Start with 6-month periods; if those timeout, try quarterly (3 months)
- The `--sort tons_sum:desc` ensures you see the highest volume lanes first in each period
- Keep the same `--group-by` parameters across all queries for easier comparison
