---
title: Calculating managed transport order percentages
when: When you need to calculate the percentage of managed vs unmanaged transport orders for a broker, either as a simple total or as a time-series trend with visualizations
---

# Calculating managed transport order percentages

## Overview
You can calculate the percentage of managed vs unmanaged transport orders for a broker using the `xbe view transport-orders list` command with JSON output and filtering.

## Simple percentage calculation

### Step 1: Get all transport orders for a date range
```bash
xbe view transport-orders list \
  --broker <broker-id> \
  --start-on <start-date> \
  --end-on <end-date> \
  --json > /tmp/transport_orders.json
```

### Step 2: Calculate percentages with jq
```bash
jq -r '
  (.rows | length) as $total |
  (.rows | map(select(.is_managed == true)) | length) as $managed |
  (.rows | map(select(.is_managed == false)) | length) as $unmanaged |
  ($managed / $total * 100) as $pct_managed |
  "Total: \($total)\nManaged: \($managed) (\($pct_managed | floor)%)\nUnmanaged: \($unmanaged) (\(100 - ($pct_managed | floor))%)"
' /tmp/transport_orders.json
```

## Time-series trend analysis with visualization

For analyzing trends over time (e.g., weekly managed percentage growth), you can aggregate by date and create visualizations.

### Step 1: Get transport orders with pickup dates
```bash
xbe view transport-orders list \
  --broker <broker-id> \
  --start-on <start-date> \
  --end-on <end-date> \
  --json > /tmp/transport_orders_by_date.json
```

### Step 2: Aggregate by week and create visualization
```python
import json
from datetime import datetime
from collections import defaultdict

# Read the JSON data
with open('/tmp/transport_orders_by_date.json', 'r') as f:
    data = json.load(f)

# Aggregate by week
weekly_data = defaultdict(lambda: {'managed': 0, 'unmanaged': 0, 'dates': []})

for row in data['rows']:
    pickup_date = row['pickup_date']
    is_managed = row['is_managed']
    
    # Parse date and get ISO week
    date_obj = datetime.strptime(pickup_date, '%Y-%m-%d')
    year = date_obj.isocalendar()[0]
    week = date_obj.isocalendar()[1]
    week_key = f"{year}-W{week:02d}"
    
    weekly_data[week_key]['dates'].append(date_obj)
    
    if is_managed:
        weekly_data[week_key]['managed'] += 1
    else:
        weekly_data[week_key]['unmanaged'] += 1

# Calculate percentages and create ASCII chart
print("\nManaged Transport Order Percentage by Week")
print("=" * 90)
print(f"{'Week':<12} {'Date Range':<14} {'Managed %':<10} {'Total':<10} Bar")
print("-" * 90)

for week_key in sorted(weekly_data.keys()):
    managed = weekly_data[week_key]['managed']
    unmanaged = weekly_data[week_key]['unmanaged']
    total = managed + unmanaged
    pct_managed = (managed / total * 100) if total > 0 else 0
    
    # Get date range for the week
    dates = weekly_data[week_key]['dates']
    start_date = min(dates).strftime('%m/%d')
    end_date = max(dates).strftime('%m/%d')
    
    # Create ASCII bar (scale to fit terminal)
    bar_length = int(pct_managed / 100 * 40)
    bar = '█' * bar_length
    
    print(f"{week_key:<12} {start_date}-{end_date:<8} {pct_managed:>6.1f}%    {total:>6,}     {bar}")

print("=" * 90)

# Summary statistics
all_managed = sum(w['managed'] for w in weekly_data.values())
all_unmanaged = sum(w['unmanaged'] for w in weekly_data.values())
all_total = all_managed + all_unmanaged
overall_pct = (all_managed / all_total * 100) if all_total > 0 else 0

print(f"\nOverall Summary:")
print(f"  Total Orders: {all_total:,}")
print(f"  Managed: {all_managed:,} ({overall_pct:.1f}%)")
print(f"  Unmanaged: {all_unmanaged:,} ({100-overall_pct:.1f}%)")
```

## Filtering to specific weeks

You can filter the weekly output to start from a specific week by adding a condition:

```python
# In the output loop, add a filter
for week_key in sorted(weekly_data.keys()):
    if week_key < '<start-week>':  # e.g., '2025-W39'
        continue
    # ... rest of the code
```

## Notes
- The `is_managed` field indicates whether a transport order is managed (true) or unmanaged (false)
- Use `--start-on` and `--end-on` to filter by date range, or omit them to use the default "Today & Tomorrow" window
- Each █ character in the bar chart represents approximately 2.5% when using a 40-character scale
- ISO weeks run Monday-Sunday and may span across year boundaries (e.g., 2026-W01 includes days from late December 2025)
