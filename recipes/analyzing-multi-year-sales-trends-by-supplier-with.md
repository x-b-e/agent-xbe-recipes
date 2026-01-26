---
title: Analyzing multi-year sales trends by supplier with seasonal patterns
when: When you need to analyze a material supplier's sales performance across multiple years, including seasonal patterns, year-over-year comparisons, and identifying changes in business trajectory
---

# Analyzing multi-year sales trends by supplier with seasonal patterns

## Overview
This recipe shows how to analyze a material supplier's sales performance across multiple years, breaking down by month to identify seasonal patterns, year-over-year changes, and shifts in business trajectory.

## Steps

### 1. Find the material supplier ID
First, identify the supplier you want to analyze:

```bash
xbe view material-suppliers list --json | jq '.[] | select(.name | test("<supplier-name>"; "i"))'
```

Extract the `id` field from the result.

### 2. Query material transactions grouped by year and month
Use `material-transaction-summary` to get tonnage by year and month:

```bash
xbe summarize material-transaction-summary create \
  --filter material_supplier=<material-supplier-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by year,month \
  --json
```

This returns data structured by year and month with tonnage totals.

### 3. Process and restructure the data
The raw output needs processing. Save to a file and use jq to create a year-by-month matrix:

```bash
xbe summarize material-transaction-summary create \
  --filter material_supplier=<material-supplier-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by year,month \
  --json > /tmp/supplier_sales.json

# Transform into year-by-month structure
jq '[.[] | {year: .year, month: .month, tons: (.tons_outbound // 0)}] | 
    group_by(.year) | 
    map({year: .[0].year, months: map({month: .month, tons: .tons})})' \
  /tmp/supplier_sales.json
```

### 4. Create comprehensive comparative analysis with Python
Use Python to calculate:
- Year-over-year growth rates
- Seasonal patterns (which months are strongest/weakest)
- Comparison to historical averages
- Peak season performance
- Volatility/consistency metrics

```python
import json
import statistics

# Load data
with open('/tmp/supplier_sales.json') as f:
    raw_data = json.load(f)

# Restructure into year -> [month_values] dictionary
data = {}
for entry in raw_data:
    year = entry['year']
    month = entry['month']
    tons = entry.get('tons_outbound', 0)
    
    if year not in data:
        data[year] = [0] * 12
    data[year][month - 1] = tons

# Calculate year-over-year changes
print("YEAR-OVER-YEAR TRENDS:")
for year in sorted(data.keys()):
    total = sum(data[year])
    if year > min(data.keys()):
        prev_total = sum(data[year-1])
        change_pct = ((total - prev_total) / prev_total) * 100
        print(f"{year}: {total:,} tons ({change_pct:+.1f}%)")
    else:
        print(f"{year}: {total:,} tons (baseline)")

# Compare latest year to historical average by month
latest_year = max(data.keys())
historical_years = [y for y in data.keys() if y < latest_year]

print(f"\n{latest_year} vs HISTORICAL AVERAGE BY MONTH:")
for month_idx in range(12):
    latest_val = data[latest_year][month_idx]
    hist_avg = statistics.mean([data[y][month_idx] for y in historical_years])
    diff = latest_val - hist_avg
    pct_change = (diff / hist_avg * 100) if hist_avg > 0 else 0
    
    month_name = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 
                  'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'][month_idx]
    print(f"{month_name}: {latest_val:,} vs {hist_avg:,.0f} ({pct_change:+.1f}%)")

# Analyze seasonal patterns
print("\nSEASONAL PATTERNS:")
for season_name, months in [
    ("Winter (Jan-Feb)", [0, 1]),
    ("Spring (Mar-May)", [2, 3, 4]),
    ("Summer (Jun-Aug)", [5, 6, 7]),
    ("Fall (Sep-Nov)", [8, 9, 10]),
    ("December", [11])
]:
    print(f"\n{season_name}:")
    for year in sorted(data.keys()):
        season_total = sum(data[year][i] for i in months)
        year_total = sum(data[year])
        pct_of_year = (season_total / year_total * 100) if year_total > 0 else 0
        print(f"  {year}: {season_total:,} tons ({pct_of_year:.1f}% of year)")
```

## Key Insights to Extract

1. **Volume trends**: Is the business growing, declining, or cyclical?
2. **Seasonal strength**: Which seasons/months are strongest? Is this consistent across years?
3. **Pattern changes**: Did the latest year follow typical patterns or show departures?
4. **Recovery/decline signals**: Are there signs of business acceleration or deceleration?
5. **Concentration metrics**: What % of annual volume comes in peak months?

## Related Recipes
- `analyzing-material-volume-trends-by-time-period.md` - For simpler time-based volume analysis
- `comparing-metrics-across-multiple-years-in-tabular.md` - For tabular year-over-year comparisons
- `comparing-seasonality-patterns-between-geographic-markets.md` - For comparing seasonal patterns across entities
