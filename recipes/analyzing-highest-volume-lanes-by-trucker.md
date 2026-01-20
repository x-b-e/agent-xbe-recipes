---
title: Analyzing highest volume lanes by trucker
when: When you need to identify top lanes broken down by trucker to see which truckers handle the highest volumes on which routes for a broker
---

# Analyzing highest volume lanes by trucker

## Problem
You need to analyze lanes by both trucker and origin/destination to see which truckers handle the highest volumes on which specific routes.

## Solution
Use `xbe summarize lane-summary create` with `--group-by trucker,origin,destination` to get lane data broken down by trucker, then sort by tonnage.

### Find the broker ID first
```bash
xbe view broker list --name "<broker-name>"
```

### Generate lane summary by trucker
```bash
xbe summarize lane-summary create \
  --group-by trucker,origin,destination \
  --filter broker=<broker-id> \
  --filter transaction_at_min=<start-date> \
  --filter transaction_at_max=<end-date> \
  --sort tons_sum:desc \
  --limit <number> \
  --json
```

### Key flags
- `--group-by trucker,origin,destination` - Groups by trucker AND lane (origin/destination pair)
- `--sort tons_sum:desc` - Sorts results by total tonnage in descending order
- `--limit <number>` - Limits results to top N lanes (optional but useful for high-volume data)
- `--json` - Returns structured JSON output for further processing or visualization

### Example output structure
The JSON response includes:
- `headers`: Column names (e.g., trucker_name, origin_name, destination_name, cycle_count, tons_sum)
- `values`: Array of row data as arrays
- `rows`: Array of row data as objects (easier to work with)

### Date range tips
- For "past year": Use transaction_at_min/max with dates approximately 1 year apart
- Transaction dates are in ISO 8601 format (YYYY-MM-DDTHH:MM:SSZ)
- The API accepts various date formats but ISO 8601 is most reliable

## Notes
- Default metrics include: cycle_count, material_transaction_count, cycle_minutes_median, calculated_travel_minutes_median, tons_sum
- Use `--metrics` flag to customize which metrics are returned
- The `--sort` flag defaults to `material_transaction_count:desc` if not specified
- Results can be piped to jq for additional processing or used to create visualizations
