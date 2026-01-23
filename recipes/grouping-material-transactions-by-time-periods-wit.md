---
title: Grouping material transactions by time periods with category filtering
when: When you need to analyze material transaction volumes over time (monthly, yearly) for a broad material category rather than specific material types, especially for trend analysis and seasonal pattern identification
---

## Overview
Use `xbe summarize material-transaction-summary create` with both material category filtering and time-based grouping to analyze volume trends over time.

## Key Pattern
- Filter by `material_type_fully_qualified_name_base` to aggregate all variants within a category
- Combine with `--group-by year,month` for time-based breakdown
- Use `--limit` to ensure you get all time periods (default may truncate)

## Example
```bash
# Get monthly asphalt mixture volumes over multiple years
xbe summarize material-transaction-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --filter material_type_fully_qualified_name_base="Asphalt Mixture" \
  --group-by year,month \
  --metrics tons_sum \
  --limit 100
```

This returns data sorted by volume (highest first) with columns: `year`, `month`, `tons_sum`

## Why This Works
- `material_type_fully_qualified_name_base` captures the category level (e.g., "Asphalt Mixture", "Aggregate")
- This aggregates across all specific variants (e.g., all "Asphalt Mixture:HMA:SB*" types)
- Grouping by `year,month` provides time series breakdown
- Higher `--limit` ensures all time periods are returned (not just top N by volume)

## Related Patterns
- To see which specific material types contribute most, remove the time grouping and group by `material_type_name` instead
- To get overall totals, omit the `--group-by` parameter entirely
