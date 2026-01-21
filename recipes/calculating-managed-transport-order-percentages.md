---
title: Calculating managed transport order percentages
when: When you need to calculate the percentage of managed vs unmanaged transport orders for a broker, either as a simple total or as a time-series trend with visualizations
---

# Calculating managed transport order percentages

## Basic percentage calculation

To get the current percentage of managed orders for a broker:

```bash
# Get today's managed percentage
xbe view transport-orders list --broker <broker-id> \
  --start-on <date> --end-on <date> --json | \
  jq -r '[.rows[] | select(.is_managed == true)] | length as $managed | 
    (input | length) as $total | 
    ($managed / $total * 100 | tostring + "%")'
```

## Weekly time-series analysis

To analyze managed order trends over time:

```bash
# 1. Export all orders with managed status and pickup dates to JSON
xbe view transport-orders list --broker <broker-id> \
  --start-on <start-date> --end-on <end-date> --json > /tmp/managed_by_date.json

# 2. Process with Python to group by week and calculate percentages
cat > /tmp/process_weekly.py << 'EOF'
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

# Print ASCII chart (optionally filter to specific weeks)
print("\nManaged Transport Order Percentage by Week")
print("=" * 90)
print(f"{'Week':<12} {'Date Range':<14} {'Managed %':<10} {'Total':<10} Bar")
print("-" * 90)

for result in results:
    # Optional: filter to start from specific week
    # if result['week'] < '2025-W39':
    #     continue
    
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

print(f"\nOverall Summary:")
print(f"  Total Orders: {all_total:,}")
print(f"  Managed: {all_managed:,} ({overall_pct:.1f}%)")
print(f"  Unmanaged: {all_unmanaged:,} ({100-overall_pct:.1f}%)")
EOF

python3 /tmp/process_weekly.py
```

## Key points

- Use `--is-managed true/false` flag to filter by managed status
- The `is_managed` field in JSON output indicates whether each order is managed
- Group by ISO week using `date_obj.isocalendar()[0]` (year) and `[1]` (week number)
- Track date ranges by collecting all dates in each week and using min/max
- Filter results to specific week ranges by comparing week_key strings (e.g., `>= '2025-W39'`)
- ASCII bars help visualize trends in the terminal (each █ can represent ~2.5%)

## Finding the broker ID

If you need to find a broker's ID by name first:

```bash
xbe view brokers list --q "<broker-name>" --json | jq '.rows[] | {id, name}'
```
