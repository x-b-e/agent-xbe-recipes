---
title: Calculating managed transport order percentages
when: When you need to calculate the percentage of managed vs unmanaged transport orders for a broker, or when you need to calculate driver confirmation percentages for planned transport orders, either as simple totals or broken down by office with visualizations
---

# Calculating managed transport order percentages

## Problem
You need to calculate what percentage of transport orders are managed vs unmanaged, or calculate driver confirmation percentages for planned orders.

## Solution

### Simple managed order percentage (today)

1. Get total orders and managed orders for today:
```bash
xbe view transport-orders list --broker <broker-id> --start-on <today-date> --end-on <today-date> --json | jq '{total: length, managed: [.[] | select(.is_managed == true)] | length} | .managed / .total * 100'
```

### Driver confirmation percentage (simple)

1. Query transport plan driver data:
```bash
xbe query transport-plan-drivers \
  --broker <broker-id> \
  --start-on <start-date> \
  --end-on <end-date> \
  --select project_transport_plan_driver_count,project_transport_plan_driver_confirmation_count \
  --json
```

2. Calculate the percentage:
```bash
xbe query transport-plan-drivers ... --json | jq '
  .[] | 
  (.project_transport_plan_driver_confirmation_count / .project_transport_plan_driver_count * 100)
'
```

### Driver confirmation percentage by office (with visualization)

1. Query with office grouping:
```bash
xbe query transport-plan-drivers \
  --broker <broker-id> \
  --start-on <start-date> \
  --end-on <end-date> \
  --select project_transport_plan_driver_count,project_transport_plan_driver_confirmation_count \
  --group-by project_office_name \
  --json
```

This also works for comparing against transport orders:
```bash
xbe query transport-plan-drivers \
  --broker <broker-id> \
  --start-on <start-date> \
  --end-on <end-date> \
  --select transport_order_count,project_transport_plan_driver_count,project_transport_plan_driver_confirmation_count \
  --group-by project_office_name
```

2. Process and visualize with Python:
```python
import sys
import json

# Read JSON from stdin or variable
data = json.loads(sys.stdin.read())

results = []
for row in data:
    office = row['project_office_name']
    drivers = float(row['project_transport_plan_driver_count'])
    confirmations = float(row['project_transport_plan_driver_confirmation_count'])
    
    pct = (confirmations / drivers * 100) if drivers > 0 else 0
    
    results.append({
        'office': office,
        'drivers': int(drivers),
        'confirmations': int(confirmations),
        'pct': pct
    })

# Sort by confirmation percentage descending
results.sort(key=lambda x: x['pct'], reverse=True)

print(f"{'Office':<20} {'Drivers':<10} {'Confirmed':<12} {'Confirm %':<12}")
print("-" * 60)

for r in results:
    # Create mini bar chart
    bar_length = int(r['pct'] / 10)
    bar = '█' * bar_length if bar_length > 0 else ''
    
    print(f"{r['office']:<20} {r['drivers']:<10} {r['confirmations']:<12} {r['pct']:>5.1f}%  {bar}")

# Print totals
total_drivers = sum(r['drivers'] for r in results)
total_confirmations = sum(r['confirmations'] for r in results)
total_pct = (total_confirmations / total_drivers * 100) if total_drivers > 0 else 0
print("=" * 60)
print(f"{'TOTAL':<20} {total_drivers:<10} {total_confirmations:<12} {total_pct:>5.1f}%")
```

### Time-series trend analysis

1. Query with date grouping:
```bash
xbe query transport-plan-drivers \
  --broker <broker-id> \
  --start-on <start-date> \
  --end-on <end-date> \
  --select project_transport_plan_driver_count,project_transport_plan_driver_confirmation_count \
  --group-by date \
  --json
```

2. Process for trend visualization (can combine with gnuplot or similar).

## Key points

- Use `--group-by project_office_name` to break down confirmation rates by office
- Use `--group-by date` for time-series analysis
- Sort results by percentage to identify best and worst performing offices
- Visual bar charts help quickly identify patterns
- Always handle division by zero when calculating percentages
