---
title: Calculating managed transport order percentages
when: When you need to calculate the percentage of managed vs unmanaged transport orders for a broker, either as a simple total or as a time-series trend
---

# Calculating managed transport order percentages

## Simple percentage for today

To get the percentage of managed transport orders for a broker on today's date:

```bash
xbe view transport-orders list --broker <broker-id> \
  --start-on <date> --end-on <date> --json | \
  jq -r '.rows | group_by(.is_managed) | 
    map({is_managed: .[0].is_managed, count: length}) | 
    "Managed: \(map(select(.is_managed == true) | .count)[0] // 0), Unmanaged: \(map(select(.is_managed == false) | .count)[0] // 0)"'
```

## Time-series analysis with weekly trends

For a comprehensive weekly trend analysis, use a Python script to process the data:

### Step 1: Export transport orders to JSON

```bash
xbe view transport-orders list --broker <broker-id> \
  --start-on <start-date> --end-on <end-date> \
  --json --limit 200000 > /tmp/transport_orders.json
```

### Step 2: Aggregate by date and managed status

Use jq to create a summary grouped by pickup date and managed status:

```bash
jq '{rows: [.rows | group_by(.pickup_date, .is_managed) | .[] | 
  {pickup_date: .[0].pickup_date, is_managed: .[0].is_managed, transport_order_count: length}]}' \
  /tmp/transport_orders.json > /tmp/managed_by_date.json
```

### Step 3: Calculate weekly percentages with Python

```python
import json
from datetime import datetime
from collections import defaultdict

# Read the JSON data
with open('/tmp/managed_by_date.json', 'r') as f:
    data = json.load(f)

# Process rows to calculate weekly stats
weekly_data = defaultdict(lambda: {'managed': 0, 'unmanaged': 0, 'dates': []})

for row in data['rows']:
    pickup_date = row['pickup_date']
    is_managed = row['is_managed']
    count = int(row['transport_order_count'])
    
    # Parse date and get ISO week
    date_obj = datetime.strptime(pickup_date, '%Y-%m-%d')
    year = date_obj.isocalendar()[0]
    week = date_obj.isocalendar()[1]
    week_key = f"{year}-W{week:02d}"
    
    weekly_data[week_key]['dates'].append(date_obj)
    
    if is_managed:
        weekly_data[week_key]['managed'] += count
    else:
        weekly_data[week_key]['unmanaged'] += count

# Calculate percentages and create output
print("\nManaged Transport Order Percentage by Week")
print("=" * 90)
print(f"{'Week':<12} {'Date Range':<14} {'Managed %':<10} {'Total':<10} Bar")
print("-" * 90)

for week_key in sorted(weekly_data.keys()):
    managed = weekly_data[week_key]['managed']
    unmanaged = weekly_data[week_key]['unmanaged']
    total = managed + unmanaged
    pct_managed = (managed / total * 100) if total > 0 else 0
    
    # Get date range
    dates = weekly_data[week_key]['dates']
    start_date = min(dates).strftime('%m/%d')
    end_date = max(dates).strftime('%m/%d')
    
    # Optional: filter to start from specific week
    # if week_key < '<start-week>':  # e.g., '2025-W39'
    #     continue
    
    # Create ASCII bar (max 40 characters)
    bar_length = int(pct_managed / 100 * 40)
    bar = '█' * bar_length
    
    print(f"{week_key:<12} {start_date}-{end_date:<10} {pct_managed:>6.1f}%    {total:>6,}     {bar}")

print("=" * 90)
```

## Tips

- The `--is-managed` filter accepts `true` or `false` to filter to only managed or unmanaged orders
- Use `--limit 200000` for large date ranges to ensure you get all results
- Each █ in the ASCII chart represents approximately 2.5% when using a max bar length of 40
- To show date ranges, track all dates within each week and use `min(dates)` and `max(dates)`
- To filter the output to start from a specific week, add a condition like `if week_key < '2025-W39': continue`
