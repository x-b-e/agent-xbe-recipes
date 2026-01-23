---
title: Analyzing material volume trends by time period
when: When you need to analyze material transaction volumes (tonnage) over time periods (monthly, yearly) to identify seasonal patterns, peak periods, or year-over-year trends for a specific material category or broker
---

# Analyzing material volume trends by time period

## Context
When analyzing material sales or usage patterns, you often need to see how volumes change over time. The `xbe summarize material-transaction-summary` command can group data by time dimensions to reveal seasonal patterns and trends.

## Steps

### 1. Filter by material category and time range

To analyze a specific material category (like "Asphalt Mixture") over a date range:

```bash
xbe summarize material-transaction-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --filter material_type_fully_qualified_name_base="<material-category>" \
  --group-by year,month \
  --metrics tons_sum \
  --limit 100
```

### 2. Understanding the filter syntax

- **material_type_fully_qualified_name_base**: Filters for all materials that start with this base category
  - Example: `"Asphalt Mixture"` matches "Asphalt Mixture:HMA:SB03", "Asphalt Mixture:HMA:SB62", etc.
  - This captures all variants of a material category

### 3. Grouping by time dimensions

- **Single dimension**: `--group-by year` or `--group-by month`
- **Multiple dimensions**: `--group-by year,month` groups by both, showing each month within each year
- **Other time dimensions**: `week`, `quarter` are also available

### 4. Interpreting results

The output shows tonnage aggregated by the specified time periods:

```
year    month  tons_sum
2025.0  8.0    259644.57
2023.0  9.0    249599.57
2023.0  8.0    244552.93
```

**Note**: Results are typically sorted by volume (tons_sum) descending, not chronologically. You may need to reorder manually or with post-processing to see chronological trends.

### 5. Handling large date ranges

If you encounter timeouts with large date ranges:
- Start with a smaller date range (e.g., one year)
- Use more specific material filters
- Consider grouping by larger time periods (year vs month)

## Example: Analyzing seasonal patterns

```bash
# Get asphalt mixture tonnage by month for 3 years
xbe summarize material-transaction-summary create \
  --filter broker=<broker-id> \
  --filter date_min=2023-01-01 \
  --filter date_max=2025-12-31 \
  --filter material_type_fully_qualified_name_base="Asphalt Mixture" \
  --group-by year,month \
  --metrics tons_sum \
  --limit 100
```

This reveals:
- **Peak months**: June-October typically show highest volumes (paving season)
- **Low months**: January-February show significantly lower volumes (winter)
- **Year-over-year trends**: Compare the same month across years to see growth or decline

## Related patterns

- To see all material types first: Remove the material filter and `--group-by material_type` to identify what materials are available
- To compare multiple brokers: Run separate queries and compare results
- To get totals without time breakdown: Remove `--group-by` parameters for aggregate totals
