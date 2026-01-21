---
title: Calculating managed transport order percentages
when: When you need to calculate the percentage of managed vs unmanaged transport orders for a broker, either as a simple total or as a time-series trend
---

# Calculating managed transport order percentages

## Simple percentage calculation

To get the overall percentage of managed vs unmanaged orders:

```bash
# Get today's managed order percentage
xbe summarize transport-order-efficiency-summary create \
  --group-by is_managed \
  --filter broker=<broker-id> \
  --filter pickup_date_min=<date> \
  --filter pickup_date_max=<date> \
  --json
```

The output includes `transport_order_count` for each `is_managed` value (true/false).

## Weekly trend analysis

To analyze how the managed percentage has changed over time:

```bash
# Get data grouped by both date and managed status
xbe summarize transport-order-efficiency-summary create \
  --group-by pickup_date,is_managed \
  --filter broker=<broker-id> \
  --filter pickup_date_min=<start-date> \
  --filter pickup_date_max=<end-date> \
  --json > /tmp/managed_by_date.json
```

## Processing weekly trends with Python

Use Python to aggregate daily data into weekly percentages:

```python
import json
from datetime import datetime
from collections import defaultdict

with open('/tmp/managed_by_date.json', 'r') as f:
    data = json.load(f)

weekly_data = defaultdict(lambda: {'managed': 0, 'unmanaged': 0})

for row in data['rows']:
    pickup_date = row['pickup_date']
    is_managed = row['is_managed']
    count = int(row['transport_order_count'])
    
    # Get ISO week
    date_obj = datetime.strptime(pickup_date, '%Y-%m-%d')
    year = date_obj.isocalendar()[0]
    week = date_obj.isocalendar()[1]
    week_key = f"{year}-W{week:02d}"
    
    if is_managed:
        weekly_data[week_key]['managed'] += count
    else:
        weekly_data[week_key]['unmanaged'] += count

# Calculate percentages
for week_key in sorted(weekly_data.keys()):
    managed = weekly_data[week_key]['managed']
    unmanaged = weekly_data[week_key]['unmanaged']
    total = managed + unmanaged
    pct_managed = (managed / total * 100) if total > 0 else 0
    print(f"{week_key}: {pct_managed:.1f}% ({total} orders)")
```

## Key insights

- Group by both `pickup_date` and `is_managed` to get time-series data
- Aggregate daily data into weekly buckets using ISO week numbers
- Calculate percentage as: `(managed / total) * 100`
- Consider creating visualizations (ASCII bar charts or HTML) for stakeholder presentations
