---
title: Calculating managed transport order percentages
when: When you need to calculate the percentage of managed vs unmanaged transport orders for a broker, either as a simple total or as a time-series trend with visualizations
---

# Calculating managed transport order percentages

## Basic percentage for today

To get today's managed transport order percentage:

```bash
xbe summarize transport-order-efficiency-summary create \
  --group-by is_managed \
  --filter broker=<broker-id> \
  --filter pickup_date_min=<today-date> \
  --filter pickup_date_max=<today-date> \
  --json | jq -r '
    .rows | 
    group_by(.is_managed) | 
    map({is_managed: .[0].is_managed, count: (map(.transport_order_count | tonumber) | add)}) | 
    (map(select(.is_managed == true)) | .[0].count // 0) as $managed |
    (map(.count) | add) as $total |
    "Managed: \($managed)/\($total) (\(($managed / $total * 100 * 10 | round) / 10)%)"
  '
```

## Time-series analysis with weekly grouping

For trend analysis over time, group by date and managed status, then post-process to calculate weekly percentages:

```bash
# Step 1: Get raw data grouped by date and managed status
xbe summarize transport-order-efficiency-summary create \
  --group-by pickup_date,is_managed \
  --filter broker=<broker-id> \
  --filter pickup_date_min=<start-date> \
  --filter pickup_date_max=<end-date> \
  --json > /tmp/managed_by_date.json
```

```python
# Step 2: Process into weekly percentages with visualization
import json
from datetime import datetime
from collections import defaultdict

with open('/tmp/managed_by_date.json', 'r') as f:
    data = json.load(f)

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

# Calculate percentages and create visualization
print("\nManaged Transport Order Percentage by Week")
print("=" * 90)
print(f"{'Week':<12} {'Date Range':<14} {'Managed %':<10} {'Total':<10} Bar")
print("-" * 90)

for week_key in sorted(weekly_data.keys()):
    managed = weekly_data[week_key]['managed']
    unmanaged = weekly_data[week_key]['unmanaged']
    total = managed + unmanaged
    pct_managed = (managed / total * 100) if total > 0 else 0
    
    dates = weekly_data[week_key]['dates']
    start_date = min(dates).strftime('%m/%d')
    end_date = max(dates).strftime('%m/%d')
    
    # Create ASCII bar (max 40 characters)
    bar_length = int(pct_managed / 100 * 40)
    bar = '█' * bar_length
    
    print(f"{week_key:<12} {start_date}-{end_date:<14} {pct_managed:>6.1f}%    {total:>6,}     {bar}")

print("=" * 90)
```

## Filtering by office or other dimensions

To analyze specific offices, filter for all offices then post-process:

```bash
# Get data for all offices
xbe summarize transport-order-efficiency-summary create \
  --group-by pickup_date,is_managed,project_office \
  --filter broker=<broker-id> \
  --filter pickup_date_min=<start-date> \
  --filter pickup_date_max=<end-date> \
  --json > /tmp/all_offices_managed.json
```

```python
# Then filter in Python for specific office
for row in data['rows']:
    if row.get('project_office_name') != '<office-name>':
        continue
    # ... process as above
```

## Notes

- The `--filter project_office=<office-name>` parameter expects an office ID (integer), not a name (string), which causes a 500 error. Filter by office name in post-processing instead.
- Use ISO week calculations (`isocalendar()`) for consistent weekly grouping across year boundaries.
- ASCII bar charts provide quick visual feedback without requiring external tools.
- For summary statistics across all weeks, sum up the managed/unmanaged counts and calculate the overall percentage.
