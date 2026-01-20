---
title: Analyze highest volume lanes by trucker for a broker
when: When you need to analyze lane volumes broken down by trucker for a specific broker over a time period
---

To get the highest volume lanes with trucker breakdown:

## Step 1: Find the broker ID
```bash
xbe view brokers list --company-name "<broker-name>" --json
```

## Step 2: Create lane summary grouped by origin, destination, and trucker
```bash
xbe do lane-summary create \
  --group-by origin,destination,trucker \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --metrics tons_sum,material_transaction_count \
  --sort tons_sum:desc \
  --limit <number-of-results>
```

## Key parameters:
- `--group-by origin,destination,trucker`: Groups by all three dimensions to show which trucker handles each lane
- `--metrics tons_sum,material_transaction_count`: Shows both total tonnage and number of loads
- `--sort tons_sum:desc`: Orders by highest tonnage first
- `--filter broker=<id>`: Filters to specific broker
- Date filters use format `YYYY-MM-DD`

## Optional: Check actual date range in data
```bash
# Earliest date
xbe do material-transaction-summary create \
  --group-by date \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --metrics tons_sum \
  --sort date:asc \
  --limit 1 \
  --json | jq -r '.rows[0].date'

# Latest date
xbe do material-transaction-summary create \
  --group-by date \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --metrics tons_sum \
  --sort date:desc \
  --limit 1 \
  --json | jq -r '.rows[0].date'
```
