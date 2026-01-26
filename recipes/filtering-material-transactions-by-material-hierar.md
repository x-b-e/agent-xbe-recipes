---
title: Filtering material transactions by material category
when: When you need to filter material transaction summaries by a broad material category (like 'Asphalt Mixture', 'Aggregate', 'Concrete') rather than individual material types
---

# Filtering material transactions by material category

## Problem
You need to analyze material transactions for a broad category (e.g., all asphalt mixtures, all aggregates) rather than individual material types. The category might include many variants with different IDs.

## Solution
Use the `material_type_fully_qualified_name_base` filter to filter by the base material category name directly:

```bash
xbe summarize material-transaction-summary create \
  --group-by month \
  --filter broker=<broker-id> \
  --filter year=<year> \
  --filter "material_type_fully_qualified_name_base=<category-name>" \
  --json
```

## Common base category names
- "Asphalt Mixture" - captures all HMA/asphalt products
- "Aggregate" - captures all aggregate types
- "Concrete" - captures all ready-mix concrete

## Example: Monthly asphalt sales
```bash
xbe summarize material-transaction-summary create \
  --group-by month \
  --filter broker=<broker-id> \
  --filter year=<year> \
  --filter "material_type_fully_qualified_name_base=Asphalt Mixture" \
  --json
```

## Why this works
The `material_type_fully_qualified_name_base` filter matches all material types whose base name matches the specified category, automatically capturing all variants without needing to query hierarchies or enumerate individual material type IDs.

## Getting totals without grouping
To get a single total across all periods, use an empty group-by:

```bash
xbe summarize material-transaction-summary create \
  --group-by "" \
  --filter broker=<broker-id> \
  --filter year=<year> \
  --filter "material_type_fully_qualified_name_base=Asphalt Mixture" \
  --json
```
