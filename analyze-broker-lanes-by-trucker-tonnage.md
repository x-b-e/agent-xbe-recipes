---
title: Analyze broker lanes by trucker tonnage
when: User wants to analyze shipping lanes, routes, or tonnage data for a broker, grouped by trucker or carrier
---

# Analyze broker lanes by trucker tonnage

To analyze hauling lanes and tonnage for a broker:

## 1. Find the broker ID

```bash
xbe view brokers list --company-name "<broker-name>" --json
```

Extract the `id` field from the result.

## 2. Fetch material transactions for date range

```bash
xbe view material-transactions list --broker-id <broker-id> --start-date <YYYY-MM-DD> --end-date <YYYY-MM-DD> --json
```

For past year analysis, calculate dates relative to today.

## 3. Process with jq for lane analysis

Group by trucker and calculate tonnage per lane:

```bash
jq -r '
  group_by(.trucker_name) |
  map({
    trucker: .[0].trucker_name,
    total_tons: (map(.net_weight_tons // 0) | add),
    lanes: (group_by(.material_site_name + " → " + .job_site_name) |
      map({
        lane: .[0].material_site_name + " → " + .[0].job_site_name,
        tons: (map(.net_weight_tons // 0) | add),
        count: length
      }) | sort_by(-.tons))
  }) | sort_by(-.total_tons)
' transactions.json
```

## Key fields in material-transactions

- `trucker_name`: Trucking company
- `material_site_name`: Origin/source location
- `job_site_name`: Destination location
- `net_weight_tons`: Tonnage per transaction
- `created_at`: Transaction timestamp

## Tips

- Use `--limit` to control result size if needed
- Save JSON to file for complex jq processing
- Lane = unique origin-destination-trucker combination
