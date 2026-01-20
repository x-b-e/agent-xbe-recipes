---
title: Analyze highest volume lanes by origin, destination, and trucker
when: User asks for lane analysis, highest volume routes, top lanes by tons/loads, or wants to understand transportation patterns by trucker or carrier
---

# Analyzing Highest Volume Lanes by Origin, Destination, and Trucker

## When to Use
User asks for:
- "What are the highest volume lanes?"
- "Show me top routes by tons"
- "Which lanes have the most loads?"
- "Analyze transportation patterns by trucker"
- "Which truckers handle the most volume on which lanes?"

## Required Information
1. **Broker ID** - Find using `xbe view brokers list --company-name "<name>" --json`
2. **Date Range** - Use `date_min` and `date_max` filters (format: YYYY-MM-DD)
3. **Grouping** - Typically group by `origin,destination,trucker` for lane analysis

## Command Pattern

```bash
# Step 1: Find the broker ID
xbe view brokers list --company-name "<Broker Name>" --json

# Step 2: Get lane summary data grouped by origin, destination, and trucker
xbe do lane-summary create \
  --group-by origin,destination,trucker \
  --filter broker=<BROKER_ID> \
  --filter date_min=<START_DATE> \
  --filter date_max=<END_DATE> \
  --json \
  --limit 1000 | \
jq -r '.headers as $headers | 
  .values | 
  map([., $headers] | transpose | map({(.[1]): .[0]}) | add) | 
  sort_by((.tons_sum | tonumber) * -1) | 
  .[0:20] | 
  (["ORIGIN", "DESTINATION", "TRUCKER", "TONS", "CYCLES", "LOADS"]), 
  (.[] | [.origin_name, .destination_name, .trucker_name, (.tons_sum | tonumber | round), .cycle_count, .material_transaction_count]) | 
  @tsv'
```

## Key Points

- **Group-by options**: Can group by any combination of `origin`, `destination`, `trucker`, `material`, `customer`, etc.
- **Sorting**: The jq command sorts by `tons_sum` in descending order (`* -1`)
- **Top N results**: `.[0:20]` limits to top 20 lanes (adjust as needed)
- **Output format**: Tab-separated values (TSV) for easy reading
- **Metrics available**: 
  - `tons_sum` - Total tons
  - `cycle_count` - Number of cycles
  - `material_transaction_count` - Number of loads

## Alternative Groupings

```bash
# Group by origin and destination only (all truckers combined)
--group-by origin,destination

# Group by trucker only (all lanes combined)
--group-by trucker

# Include material type in analysis
--group-by origin,destination,trucker,material
```

## Tips

- Use `--limit 1000` or higher to ensure you capture all lanes before sorting
- The jq command handles the JSON-to-table transformation and sorting
- For date ranges, "past 1 year" means from one year ago to today
- The `tons_sum` field may be a string, so convert with `tonumber` before sorting
