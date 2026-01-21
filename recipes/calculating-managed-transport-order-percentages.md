---
title: Calculating managed transport order percentages
when: When you need to calculate the percentage of managed vs unmanaged transport orders for a broker, or when you need to calculate driver confirmation percentages for planned transport orders, either as simple totals or as time-series trends with visualizations
---

# Calculating managed transport order percentages

## Simple percentage calculation

To calculate what percentage of transport orders are managed vs unmanaged:

```bash
xbe summarize transport-order-efficiency-summary create \
  --group-by is_managed \
  --filter broker=<broker-id> \
  --filter pickup_date_min=<start-date> \
  --filter pickup_date_max=<end-date>
```

This returns counts grouped by `is_managed` (true/false). Calculate the percentage by dividing managed count by total count.

## Driver confirmation percentage

To calculate what percentage of planned drivers have confirmed:

```bash
xbe summarize transport-order-efficiency-summary create \
  --group-by "" \
  --filter broker=<broker-id> \
  --filter pickup_date_min=<start-date> \
  --filter pickup_date_max=<end-date> \
  --metrics transport_order_count,project_transport_plan_driver_count,project_transport_plan_driver_confirmation_count
```

Calculate the percentage as: `(project_transport_plan_driver_confirmation_count / project_transport_plan_driver_count) * 100`

## Weekly trend analysis with visualization

To see how managed percentages have trended over time:

1. Export data grouped by date and managed status:

```bash
xbe summarize transport-order-efficiency-summary create \
  --group-by pickup_date,is_managed \
  --filter broker=<broker-id> \
  --filter pickup_date_min=<start-date> \
  --filter pickup_date_max=<end-date> \
  --json > /tmp/managed_data.json
```

2. Process with Python to calculate weekly percentages:

```python
import json
from datetime import datetime
from collections import defaultdict

with open('/tmp/managed_data.json', 'r') as f:
    data = json.load(f)

weekly_data = defaultdict(lambda: {'managed': 0, 'unmanaged': 0, 'dates': []})

for row in data['rows']:
    pickup_date = row['pickup_date']
    is_managed = row['is_managed']
    count = int(row['transport_order_count'])
    
    # Get ISO week
    date_obj = datetime.strptime(pickup_date, '%Y-%m-%d')
    year = date_obj.isocalendar()[0]
    week = date_obj.isocalendar()[1]
    week_key = f"{year}-W{week:02d}"
    
    weekly_data[week_key]['dates'].append(date_obj)
    
    if is_managed:
        weekly_data[week_key]['managed'] += count
    else:
        weekly_data[week_key]['unmanaged'] += count

# Print results with ASCII bar chart
print(f"{'Week':<12} {'Date Range':<14} {'Managed %':<10} {'Total':<10} Bar")
for week_key in sorted(weekly_data.keys()):
    managed = weekly_data[week_key]['managed']
    unmanaged = weekly_data[week_key]['unmanaged']
    total = managed + unmanaged
    pct_managed = (managed / total * 100) if total > 0 else 0
    
    dates = weekly_data[week_key]['dates']
    start_date = min(dates).strftime('%m/%d')
    end_date = max(dates).strftime('%m/%d')
    
    bar_length = int(pct_managed / 100 * 40)
    bar = '█' * bar_length
    
    print(f"{week_key:<12} {start_date}-{end_date:<12} {pct_managed:>6.1f}%    {total:>6,}     {bar}")
```

## Breaking down by office or other dimensions

Add additional dimensions to the `--group-by` to see managed percentages by office, customer, etc:

```bash
xbe summarize transport-order-efficiency-summary create \
  --group-by project_office,is_managed \
  --filter broker=<broker-id> \
  --filter pickup_date_min=<start-date> \
  --filter pickup_date_max=<end-date> \
  --json
```

Then calculate percentages for each office separately in your processing script.
