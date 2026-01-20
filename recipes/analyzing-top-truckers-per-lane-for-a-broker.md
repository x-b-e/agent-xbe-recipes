---
title: Analyzing top truckers per lane for a broker
when: When you need to identify the top trucking companies operating on specific lanes (origin-destination pairs) for a broker
---

# Analyzing top truckers per lane for a broker

To find the top truckers operating on specific lanes for a broker, you need to filter by broker, date range, and the specific origin/destination pair, then group by trucker.

## Basic pattern

```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --filter origin_name="<origin-name>" \
  --filter destination_name="<destination-name>" \
  --group-by origin,destination,trucker \
  --sort tons_sum:desc \
  --limit <number>
```

## Common workflow: Top truckers for each top lane

1. First, identify the top lanes:
```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by origin,destination \
  --sort tons_sum:desc \
  --limit 10
```

2. Then, for each lane, query the top truckers:
```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --filter origin_name="<origin-name>" \
  --filter destination_name="<destination-name>" \
  --group-by origin,destination,trucker \
  --sort tons_sum:desc \
  --limit 3
```

## Understanding the results

- **Default unmanaged trucker**: Entries labeled "Default unmanaged trucker" with a contractor name in parentheses (e.g., "Default unmanaged trucker (J.M. Fahey)") indicate self-performing contractors who own their own trucks rather than hiring external trucking companies.

- **Single trucker lanes**: Many lanes may be served exclusively by one trucking company, suggesting dedicated lane assignments.

- **Cycle count vs. material transaction count**: 
  - `cycle_count` represents completed round trips
  - `material_transaction_count` represents total material deliveries
  - If cycle_count is 0 but material_transaction_count exists, the data may come from a different tracking system

## Tips

- Use exact origin and destination names from the initial lane summary query
- Names with special characters may need quotes (e.g., "O'Donnell Plant")
- Consider excluding "Default unmanaged trucker" entries if you're only interested in external trucking companies
- Increase `--limit` if you suspect there are more than 3 truckers on a lane
