---
title: Calculating managed transport order percentages
when: When you need to calculate the percentage of managed vs unmanaged transport orders for a broker, either as a simple total or as a time-series trend with visualizations
---

# Calculating managed transport order percentages

## Simple percentage calculation

To get the overall percentage of managed vs unmanaged orders:

```bash
xbe view transport-orders list \
  --broker <broker-id> \
  --start-on <start-date> \
  --end-on <end-date> \
  --json | jq -r '
    group_by(.is_managed) | 
    map({managed: .[0].is_managed, count: length}) | 
    "Managed: \(map(select(.managed == true) | .count)[0] // 0), Unmanaged: \(map(select(.managed == false) | .count)[0] // 0)"
  '
```

## Time-series trend with Python visualization

For analyzing trends over time with weekly breakdowns:

### Step 1: Export data grouped by date and managed status

```bash
xbe view transport-orders list \
  --broker <broker-id> \
  --start-on <start-date> \
  --end-on <end-date> \
  --json | jq '{
    rows: [
      group_by(.pickup_date, .is_managed) | .[] | {
        pickup_date: .[0].pickup_date,
        is_managed: .[0].is_managed,
        transport_order_count: length
      }
    ]
  }' > /tmp/managed_by_date.json
```

### Step 2: Create Python script to calculate weekly percentages with date ranges

```python
import json
from datetime import datetime
from collections import defaultdict

# Read the JSON data
with open('/tmp/managed_by_date.json', 'r') as f:
    data = json.load(f)

# Process rows to calculate weekly stats and track date ranges
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

# Calculate percentages and prepare output
results = []
for week_key in sorted(weekly_data.keys()):
    managed = weekly_data[week_key]['managed']
    unmanaged = weekly_data[week_key]['unmanaged']
    total = managed + unmanaged
    pct_managed = (managed / total * 100) if total > 0 else 0
    
    # Get date range for the week
    dates = weekly_data[week_key]['dates']
    start_date = min(dates).strftime('%m/%d')
    end_date = max(dates).strftime('%m/%d')
    
    results.append({
        'week': week_key,
        'date_range': f"{start_date}-{end_date}",
        'managed': managed,
        'unmanaged': unmanaged,
        'total': total,
        'pct_managed': pct_managed
    })

# Create ASCII bar chart
print("\nManaged Transport Order Percentage by Week")
print("=" * 90)
print(f"{'Week':<12} {'Date Range':<14} {'Managed %':<10} {'Total':<10} Bar")
print("-" * 90)

for result in results:
    if result['week'] < '<start-week>':  # Optional: filter to specific start week
        continue
        
    week = result['week']
    date_range = result['date_range']
    pct = result['pct_managed']
    total = result['total']
    
    # Create bar (max 40 characters)
    bar_length = int(pct / 100 * 40)
    bar = '█' * bar_length
    
    print(f"{week:<12} {date_range:<14} {pct:>6.1f}%    {total:>6,}     {bar}")

print("=" * 90)

# Summary stats
all_managed = sum(r['managed'] for r in results)
all_unmanaged = sum(r['unmanaged'] for r in results)
all_total = all_managed + all_unmanaged
overall_pct = (all_managed / all_total * 100) if all_total > 0 else 0

print(f"\nSummary:")
print(f"  Total Orders: {all_total:,}")
print(f"  Managed: {all_managed:,} ({overall_pct:.1f}%)")
print(f"  Unmanaged: {all_unmanaged:,} ({100-overall_pct:.1f}%)")
```

### Step 3: Run the visualization

```bash
python3 /tmp/process_weekly_managed.py
```

## Key techniques

- **ISO week calculation**: Use `date_obj.isocalendar()` to get the ISO year and week number for grouping
- **Date range tracking**: Collect all dates for each week and use `min()/max()` to find the actual start/end dates
- **ASCII bars**: Scale the bar length proportionally to the percentage (e.g., `int(pct / 100 * 40)` for max 40 chars)
- **Date formatting**: Use `strftime('%m/%d')` for compact date display in charts
