---
title: Analyzing top lanes for a broker
when: When you need to identify the highest volume lanes (origin-destination pairs) for a specific broker over a date range
---

# Analyzing top lanes for a broker

## Overview
Identify the highest volume lanes (origin-destination pairs) for a specific broker to understand their most important routes.

## Complete workflow

### Step 1: Find the broker ID
```bash
xbe view brokers list --company-name "<broker-name>"
```

This returns:
```
ID  COMPANY
<broker-id>   <broker-name>
```

### Step 2: Get top lanes with tonnage and transaction counts
```bash
xbe summarize lane-summary create \
  --group-by origin,destination \
  --filter broker=<broker-id> \
  --filter year=<year> \
  --metrics cycle_count,tons_sum \
  --limit <number>
```

## Customizing metrics
The `--metrics` flag allows you to choose which columns to include:
- `cycle_count` - number of transactions/cycles
- `tons_sum` - total tonnage
- `material_transaction_count` - number of material transactions
- `cycle_minutes_median` - median cycle time
- And many more (see `xbe summarize lane-summary create --help`)

## Example output
```
origin_name                   destination_name              cycle_count  tons_sum
<origin-1>                    <destination-1>               5837         169401.17
<origin-2>                    <destination-2>               4062         191905.17
<origin-3>                    <destination-3>               3756         148807.16
```

## Filtering by date
You can filter by:
- `--filter year=<year>` - entire year
- `--filter date_min=<date>` and `--filter date_max=<date>` - specific date range
- `--filter transaction_at_min=<timestamp>` and `--filter transaction_at_max=<timestamp>` - precise timestamps

## Related
- See "Finding broker ID by company name" for just the ID lookup step
- See "Analyzing lanes by trucker" for breaking down by trucker instead of just origin/destination
- See "Analyzing highest volume lanes by trucker" for top lanes grouped by trucker
