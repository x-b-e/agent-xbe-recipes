---
title: Calculating managed transport order percentages
when: When you need to calculate the percentage of managed vs unmanaged transport orders for a broker, or when you need to calculate driver confirmation percentages for planned transport orders, either as simple totals or broken down by office/date with time-series visualizations
---

# Calculating managed transport order percentages

## Simple percentage calculation

To get the overall percentage of managed transport orders for a broker:

```bash
xbe summarize transport-order-efficiency-summary create \
  --filter broker=<broker-id> \
  --filter pickup_date_min=<start-date> \
  --filter pickup_date_max=<end-date> \
  --metrics transport_order_count,managed_transport_order_count \
  --json
```

The result will include:
- `transport_order_count`: total orders
- `managed_transport_order_count`: orders marked as managed

Calculate percentage: `(managed_transport_order_count / transport_order_count) * 100`

## Breaking down by office

To see managed percentages by office:

```bash
xbe summarize transport-order-efficiency-summary create \
  --filter broker=<broker-id> \
  --filter pickup_date_min=<start-date> \
  --filter pickup_date_max=<end-date> \
  --group-by project_office \
  --metrics transport_order_count,managed_transport_order_count \
  --json
```

## Time-series analysis by week

To track managed transport adoption over time (e.g., weekly trends by office):

```bash
xbe summarize transport-order-efficiency-summary create \
  --filter broker=<broker-id> \
  --filter pickup_date_min=<start-date> \
  --filter pickup_date_max=<end-date> \
  --group-by pickup_date,project_office \
  --metrics transport_order_count,managed_transport_order_count,unmanaged_transport_order_count \
  --json > /tmp/office_data.json
```

Then process with Python to create weekly aggregations and visualizations:

```python
import json
from datetime import datetime
from collections import defaultdict

with open('/tmp/office_data.json', 'r') as f:
    data = json.load(f)

# Filter for specific office and aggregate by week
weekly_data = defaultdict(lambda: {'managed': 0, 'unmanaged': 0, 'dates': []})

for row in data['rows']:
    if row.get('project_office_name') != '<office-name>':
        continue
    
    pickup_date = row['pickup_date']
    managed = int(row.get('managed_transport_order_count', 0))
    unmanaged = int(row.get('unmanaged_transport_order_count', 0))
    
    # Group by ISO week
    date_obj = datetime.strptime(pickup_date, '%Y-%m-%d')
    year = date_obj.isocalendar()[0]
    week = date_obj.isocalendar()[1]
    week_key = f"{year}-W{week:02d}"
    
    weekly_data[week_key]['dates'].append(date_obj)
    weekly_data[week_key]['managed'] += managed
    weekly_data[week_key]['unmanaged'] += unmanaged

# Calculate percentages and create visualization
print(f"{'Week':<12} {'Date Range':<14} {'Managed %':<10} {'Total':<10} Bar")
print("-" * 80)

for week_key in sorted(weekly_data.keys()):
    managed = weekly_data[week_key]['managed']
    unmanaged = weekly_data[week_key]['unmanaged']
    total = managed + unmanaged
    pct_managed = (managed / total * 100) if total > 0 else 0
    
    # Get date range for the week
    dates = weekly_data[week_key]['dates']
    start_date = min(dates).strftime('%m/%d')
    end_date = max(dates).strftime('%m/%d')
    
    # ASCII bar chart (40 chars max)
    bar_length = int(pct_managed / 100 * 40)
    bar = '█' * bar_length
    
    print(f"{week_key:<12} {start_date}-{end_date:<14} {pct_managed:>6.1f}%    {total:>6,}     {bar}")
```

## Driver confirmation percentages

To calculate what percentage of planned drivers have confirmed their assignments:

```bash
xbe summarize transport-order-efficiency-summary create \
  --filter broker=<broker-id> \
  --filter pickup_date_min=<start-date> \
  --filter pickup_date_max=<end-date> \
  --metrics project_transport_plan_driver_count,project_transport_plan_driver_confirmation_count \
  --json
```

The result includes:
- `project_transport_plan_driver_count`: total planned drivers
- `project_transport_plan_driver_confirmation_count`: drivers who confirmed

Calculate percentage: `(confirmation_count / driver_count) * 100`

## Driver confirmation time-series by office

To track driver confirmation adoption over time:

```bash
xbe summarize transport-order-efficiency-summary create \
  --filter broker=<broker-id> \
  --filter pickup_date_min=<start-date> \
  --filter pickup_date_max=<end-date> \
  --group-by pickup_date,project_office \
  --metrics transport_order_count,project_transport_plan_driver_count,project_transport_plan_driver_confirmation_count \
  --json > /tmp/confirmation_data.json
```

Process similarly to managed transport percentages, using the same weekly aggregation pattern but calculating `confirmations / drivers * 100`.

## Key insights from time-series analysis

- Managed transport and driver confirmations are separate features that may be adopted at different times
- Weekly trends reveal adoption patterns (e.g., gradual ramp-up vs sudden adoption)
- ASCII bar charts provide quick visual feedback without requiring external tools
- ISO week grouping (`datetime.isocalendar()`) provides consistent week boundaries
