---
title: Comparing seasonality patterns between geographic markets
when: When you need to mathematically compare seasonality patterns (monthly volume variations) between two geographic markets or brokers to quantify differences in business volatility, peak periods, and climate impact
---

# Comparing seasonality patterns between geographic markets

When comparing how seasonal patterns differ between two markets (e.g., Kansas City vs Oklahoma City), use material transaction summaries grouped by time period combined with statistical analysis.

## Approach

1. **Extract data for each market by year** - Break large date ranges into annual queries to avoid timeouts
2. **Calculate multi-year monthly averages** - Average each month across multiple years to smooth out year-specific anomalies
3. **Compute seasonality metrics** - Use statistical measures to quantify the pattern differences

## Commands

```bash
# Get data for Market A (e.g., broker 1) - do this for each year separately
xbe summarize material-transaction-summary create \
  --filter broker=<broker-id-1> \
  --filter date_min=<year>-01-01 \
  --filter date_max=<year>-12-31 \
  --filter material_type_fully_qualified_name_base="<material-category>" \
  --group-by year,month \
  --metrics tons_sum \
  --limit 100

# Get data for Market B (e.g., broker 2) - do this for each year separately
xbe summarize material-transaction-summary create \
  --filter broker=<broker-id-2> \
  --filter date_min=<year>-01-01 \
  --filter date_max=<year>-12-31 \
  --filter material_type_fully_qualified_name_base="<material-category>" \
  --group-by year,month \
  --metrics tons_sum \
  --limit 100
```

## Statistical Analysis

Create a Python script to compute key seasonality metrics:

```python
import statistics

# Structure data as {year: [jan, feb, mar, ..., dec], ...}
market_a_data = {
    <year-1>: [<jan-tons>, <feb-tons>, ...],
    <year-2>: [<jan-tons>, <feb-tons>, ...],
    # ...
}

market_b_data = {
    <year-1>: [<jan-tons>, <feb-tons>, ...],
    <year-2>: [<jan-tons>, <feb-tons>, ...],
    # ...
}

months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']

# Calculate 3-year average for each month
market_a_avg = [sum(market_a_data[y][m] for y in years) / len(years) for m in range(12)]
market_b_avg = [sum(market_b_data[y][m] for y in years) / len(years) for m in range(12)]

# Overall averages
market_a_total_avg = sum(market_a_avg) / 12
market_b_total_avg = sum(market_b_avg) / 12

# Seasonality indices (monthly avg / overall avg * 100)
market_a_indices = [(m / market_a_total_avg * 100) for m in market_a_avg]
market_b_indices = [(m / market_b_total_avg * 100) for m in market_b_avg]

# Coefficient of Variation (measures volatility)
market_a_cv = statistics.stdev(market_a_avg) / market_a_total_avg * 100
market_b_cv = statistics.stdev(market_b_avg) / market_b_total_avg * 100

# Peak-to-trough ratio (measures seasonality extremes)
market_a_peak_trough = max(market_a_avg) / min(market_a_avg)
market_b_peak_trough = max(market_b_avg) / min(market_b_avg)

# Season comparisons (e.g., summer vs winter)
market_a_summer = (market_a_avg[5] + market_a_avg[6] + market_a_avg[7]) / 3  # Jun-Aug
market_a_winter = (market_a_avg[11] + market_a_avg[0] + market_a_avg[1]) / 3  # Dec-Feb
market_a_ratio = market_a_summer / market_a_winter

market_b_summer = (market_b_avg[5] + market_b_avg[6] + market_b_avg[7]) / 3
market_b_winter = (market_b_avg[11] + market_b_avg[0] + market_b_avg[1]) / 3
market_b_ratio = market_b_summer / market_b_winter
```

## Key Metrics to Report

1. **Coefficient of Variation (CV)**: Higher CV = more volatile/seasonal. Values >50% indicate high seasonality.
2. **Peak-to-Trough Ratio**: How many times larger is the peak month vs the trough month. Ratios >10x indicate extreme seasonality.
3. **Summer/Winter Ratio**: Specific to construction/paving - how much more active is summer vs winter
4. **Seasonality Indices**: Each month as a percentage of the annual average (100 = average month)
5. **Peak/Trough Months**: Which months see maximum and minimum activity

## Example Interpretation

- **Market A**: CV=61.9%, Peak-to-Trough=23.3x, Summer/Winter=8.7x → **Extreme seasonality** (harsh winters shut down operations)
- **Market B**: CV=35.2%, Peak-to-Trough=3.4x, Summer/Winter=2.3x → **Moderate seasonality** (milder climate allows year-round work)

## Why Split by Year

Querying 3+ years at once often causes timeouts (503 errors). Query each year separately, then combine in your analysis script.

## Related Recipes

- Use "analyzing-material-volume-trends-by-time-period.md" for single-market trend analysis
- Use "handling-lane-summary-timeouts-with-large-date-ranges.md" for timeout strategies
