---
title: Analyzing managed transport order trends over time
when: When you need to track how the percentage of managed vs unmanaged transport orders changes over time (daily, weekly, monthly)
---

# Analyzing managed transport order trends over time

To analyze trends in managed vs unmanaged transport orders over time, use the `xbe analyze lane-summaries` command with the `--group-by` flag set to include both a date dimension and `is_managed`.

## Basic pattern

```bash
xbe analyze lane-summaries \
  --broker <broker-id> \
  --start-on <start-date> \
  --end-on <end-date> \
  --group-by pickup_date,is_managed \
  --json
```

## Key points

- The `is_managed` field can be used as a grouping dimension to split orders into managed vs unmanaged categories
- Combine with date dimensions (`pickup_date`, `delivery_date`) to see trends over time
- The output includes `transport_order_count` which can be used to calculate percentages
- Use `--json` output for post-processing with Python/jq to calculate percentages and create visualizations

## Example workflow

1. Get raw data grouped by date and managed status:
```bash
xbe analyze lane-summaries \
  --broker <broker-id> \
  --start-on <start-date> \
  --end-on <end-date> \
  --group-by pickup_date,is_managed \
  --json > /tmp/managed_by_date.json
```

2. Process the data to calculate weekly/daily percentages:
```python
import json
from datetime import datetime
from collections import defaultdict

with open('/tmp/managed_by_date.json', 'r') as f:
    data = json.load(f)

# Group by date and calculate percentages
for row in data['rows']:
    date = row['pickup_date']
    is_managed = row['is_managed']
    count = int(row['transport_order_count'])
    # Your aggregation logic here
```

## Related recipes

- See "Calculating managed transport order percentages" for current/snapshot percentages
- See "Creating ASCII bar charts from lane summaries" for visualization techniques
