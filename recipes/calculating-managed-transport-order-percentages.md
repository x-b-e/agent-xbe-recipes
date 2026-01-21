---
title: Calculating managed transport order percentages
when: When you need to calculate the percentage of managed vs unmanaged transport orders for a broker, or when you need to calculate driver confirmation percentages for planned transport orders, either as simple totals or broken down by office with visualizations, or when you need to analyze managed transport adoption trends over time for specific offices
---

# Calculating managed transport order percentages

The `xbe view transport-orders list` command with `--is-managed` filter and JSON output can calculate the percentage of managed vs unmanaged transport orders. This can be done as simple totals or broken down by office with visualizations.

## Basic managed/unmanaged percentage calculation

```bash
# Get all transport orders for a broker in a date range
xbe view transport-orders list \
  --broker <broker-id> \
  --start-on <start-date> \
  --end-on <end-date> \
  --json > /tmp/transport_orders.json

# Calculate percentages with jq
jq -r '
  .rows | 
  group_by(.is_managed) | 
  map({is_managed: .[0].is_managed, count: length}) | 
  "Managed: \(map(select(.is_managed == true).count)[0] // 0)\nUnmanaged: \(map(select(.is_managed == false).count)[0] // 0)"
' /tmp/transport_orders.json
```

## Calculate managed percentage for today

```bash
# Today's date
TODAY=$(date +%Y-%m-%d)

# Get today's orders
xbe view transport-orders list \
  --broker <broker-id> \
  --start-on $TODAY \
  --end-on $TODAY \
  --json | jq -r '
  .rows | 
  group_by(.is_managed) | 
  map({is_managed: .[0].is_managed, count: length}) |
  (map(select(.is_managed == true).count)[0] // 0) as $managed |
  (map(select(.is_managed == false).count)[0] // 0) as $unmanaged |
  ($managed + $unmanaged) as $total |
  "Managed: \($managed) / \($total) (\(($managed / $total * 100) | round)%)"
'
```

## Breakdown by office with ASCII visualization

```bash
# Get all transport orders for a broker
xbe view transport-orders list \
  --broker <broker-id> \
  --start-on <start-date> \
  --end-on <end-date> \
  --json > /tmp/transport_orders_by_office.json

# Process with Python to create office breakdown with visualization
cat > /tmp/process_by_office.py << 'EOF'
import json
from collections import defaultdict

with open('/tmp/transport_orders_by_office.json', 'r') as f:
    data = json.load(f)

office_stats = defaultdict(lambda: {'managed': 0, 'total': 0})

for row in data['rows']:
    office = row.get('project_office_name', 'Unknown')
    office_stats[office]['total'] += 1
    if row.get('is_managed'):
        office_stats[office]['managed'] += 1

# Calculate percentages and sort by percentage descending
results = []
for office, stats in office_stats.items():
    pct = (stats['managed'] / stats['total'] * 100) if stats['total'] > 0 else 0
    results.append({
        'office': office,
        'managed': stats['managed'],
        'total': stats['total'],
        'pct': pct
    })

results.sort(key=lambda x: x['pct'], reverse=True)

# Print results with ASCII bar chart
print(f"{'Office':<20} {'Managed %':<12} Bar")
print("="*60)

for result in results:
    office = result['office']
    pct = result['pct']
    managed = result['managed']
    total = result['total']
    
    # Create bar (max 30 characters)
    bar_length = int(pct / 100 * 30)
    bar = '█' * bar_length
    
    print(f"{office:<20} {pct:>5.1f}% ({managed:>3}/{total:<3}) {bar}")
EOF

python3 /tmp/process_by_office.py
```

## Analyzing managed transport adoption trends over time

To understand how an office has adopted managed transport over time, you can analyze weekly trends:

```bash
# First, get material transactions data that includes transport order info
xbe view material-transactions list \
  --broker <broker-id> \
  --start-on <start-date> \
  --end-on <end-date> \
  --json > /tmp/material_txns.json

# Process to get weekly managed/unmanaged breakdown for a specific office
cat > /tmp/weekly_adoption.py << 'EOF'
import json
from datetime import datetime
from collections import defaultdict

with open('/tmp/material_txns.json', 'r') as f:
    data = json.load(f)

# Filter for specific office and process weekly stats
weekly_data = defaultdict(lambda: {'managed': 0, 'unmanaged': 0, 'dates': []})

for row in data['rows']:
    if row.get('project_office_name') != '<office-name>':
        continue
        
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
results = []
for week_key in sorted(weekly_data.keys()):
    managed = weekly_data[week_key]['managed']
    unmanaged = weekly_data[week_key]['unmanaged']
    total = managed + unmanaged
    pct_managed = (managed / total * 100) if total > 0 else 0
    
    # Get date range
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

# Print weekly trend
print(f"\n<office-name> Office - Managed Transport Order Percentage by Week")
print("="*90)
print(f"{'Week':<12} {'Date Range':<14} {'Managed %':<10} {'Total':<10} Bar")
print("-"*90)

for result in results:
    week = result['week']
    date_range = result['date_range']
    pct = result['pct_managed']
    total = result['total']
    
    # Create bar (max 40 characters)
    bar_length = int(pct / 100 * 40)
    bar = '█' * bar_length
    
    print(f"{week:<12} {date_range:<14} {pct:>6.1f}%    {total:>6,}     {bar}")

print("="*90)

# Summary stats
if results:
    all_managed = sum(r['managed'] for r in results)
    all_unmanaged = sum(r['unmanaged'] for r in results)
    all_total = all_managed + all_unmanaged
    overall_pct = (all_managed / all_total * 100) if all_total > 0 else 0

    print(f"\nSummary:")
    print(f"  Total Orders: {all_total:,}")
    print(f"  Managed: {all_managed:,} ({overall_pct:.1f}%)")
    print(f"  Unmanaged: {all_unmanaged:,} ({100-overall_pct:.1f}%)")
EOF

python3 /tmp/weekly_adoption.py
```

This weekly trend analysis is useful for:
- Understanding adoption patterns (gradual vs sudden)
- Identifying when offices started using managed transport
- Tracking sustained adoption rates over time
- Comparing adoption curves between different offices

## Driver confirmation percentages (PTP)

For driver confirmation on planned transport (PTP), use a similar approach with the `ptp_driver_confirmed` field:

```bash
# Get transport orders with driver confirmation data
xbe view transport-orders list \
  --broker <broker-id> \
  --start-on <start-date> \
  --end-on <end-date> \
  --json | jq -r '
  .rows | 
  map(select(.is_managed == true)) |
  group_by(.ptp_driver_confirmed) | 
  map({confirmed: .[0].ptp_driver_confirmed, count: length}) |
  (map(select(.confirmed == true).count)[0] // 0) as $confirmed |
  (map(select(.confirmed == false).count)[0] // 0) as $unconfirmed |
  ($confirmed + $unconfirmed) as $total |
  "Driver Confirmation Rate: \($confirmed) / \($total) (\(($confirmed / $total * 100) | round)%)"
'
```

## Notes

- The `--is-managed` filter accepts "true" or "false" as values
- For managed transport adoption trends, use material transactions data which includes transport order counts
- Driver confirmation only applies to managed (PTP) orders
- Use `--no-date-defaults` if you want to avoid the default "today & tomorrow" window
- The CLI defaults to today and tomorrow if no date range is specified
