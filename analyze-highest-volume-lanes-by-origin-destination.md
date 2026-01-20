---
title: Analyze highest volume lanes by origin, destination, and trucker
when: User asks for lane analysis, highest volume routes, top lanes by tons/loads, or wants to understand transportation patterns by trucker
---

# Analyze Highest Volume Lanes by Origin, Destination, and Trucker

## Use Case
When you need to analyze transportation lanes showing the highest volume routes broken down by origin, destination, and trucker for a broker.

## Key Command Pattern

Use `xbe do lane-summary create` with `--group-by` to aggregate lane data:

```bash
xbe do lane-summary create \
  --group-by origin,destination,trucker \
  --filter broker=<BROKER_ID> \
  --filter date_min=YYYY-MM-DD \
  --filter date_max=YYYY-MM-DD \
  --json
```

## Finding Broker ID

First, find the broker ID using:
```bash
xbe view brokers list --company-name "<PARTIAL_NAME>" --json
```

## Sorting and Formatting Results

The lane-summary returns data in a headers/values format. Use jq to process and sort:

```bash
xbe do lane-summary create --group-by origin,destination,trucker \
  --filter broker=<ID> \
  --filter date_min=YYYY-MM-DD --filter date_max=YYYY-MM-DD \
  --json --limit 1000 | jq -r '
  .headers as $headers | 
  .values | 
  map(
    [., $headers] | 
    transpose | 
    map({(.[1]): .[0]}) | 
    add
  ) | 
  # Sort by tons_sum descending
  sort_by((.tons_sum | tonumber) * -1) | 
  # Take top N results
  .[0:20] | 
  # Format as table
  (["ORIGIN", "DESTINATION", "TRUCKER", "TONS", "CYCLES", "LOADS"]),
  (.[] | [.origin_name, .destination_name, .trucker_name, (.tons_sum | tonumber | round), .cycle_count, .material_transaction_count])
  | @tsv
'
```

## Key Points

- **--group-by**: Accepts comma-separated dimensions (origin, destination, trucker, material, customer, etc.)
- **Date filters**: Use date_min and date_max for time ranges (typically past year from current date)
- **Sorting**: The tons_sum field is a string, so convert with `tonumber` before sorting
- **Output fields**: lane-summary provides tons_sum, cycle_count, material_transaction_count, and median travel/cycle times
- **Limit**: Use --limit to control result size (default may be lower)

## Alternative Groupings

You can group by different combinations:
- `--group-by origin,destination` (lanes without trucker breakdown)
- `--group-by trucker` (trucker performance across all lanes)
- `--group-by origin,destination,material` (lanes by material type)
- `--group-by customer,trucker` (customer-trucker relationships)

## Troubleshooting

- If jq fails with "cannot be negated", use `(.field | tonumber) * -1` instead of `-.field`
- If jq fails with "cannot be tsv-formatted", ensure headers are in an array: `(["HEADER1", "HEADER2"])` not `"HEADER1", "HEADER2"`
- Large result sets may need file processing instead of direct Read tool usage
