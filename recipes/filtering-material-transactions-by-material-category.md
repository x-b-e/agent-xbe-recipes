---
title: Filtering material transactions by material category
when: When you need to aggregate material transactions across all variants of a material category (e.g., all asphalt mixtures, all aggregates) rather than individual material types
---

# Filtering material transactions by material category

## Problem
You need to analyze material transactions for a broad category (like "Asphalt Mixture", "Aggregate", or "Concrete") but don't want to manually identify all the individual material type IDs that belong to that category.

## Solution
Use the `material_type_fully_qualified_name_base` filter directly in the material-transaction-summary query. This filter matches materials by their base category name.

## Example
```bash
# Get total asphalt mixture sales for a broker in 2025
xbe summarize material-transaction-summary create \
  --group-by "" \
  --filter broker=<broker-id> \
  --filter year=<year> \
  --filter "material_type_fully_qualified_name_base=Asphalt Mixture" \
  --json

# Break down by month
xbe summarize material-transaction-summary create \
  --group-by month \
  --filter broker=<broker-id> \
  --filter year=<year> \
  --filter "material_type_fully_qualified_name_base=Asphalt Mixture" \
  --json
```

## Finding available base names
If you're not sure what base name to use, query material types and look at their `base-qualified-name` field:

```bash
# Search for materials and see their base names
xbe view material-types list \
  --broker <broker-id> \
  --q "<search-term>" \
  --fields display-name,base-qualified-name \
  --json

# Group by base name to see categories
xbe view material-types list \
  --broker <broker-id> \
  --q "<search-term>" \
  --fields display-name,base-qualified-name \
  --json | jq -r 'group_by(."base-qualified-name") | map({base: .[0]."base-qualified-name", count: length}) | .[]'
```

## Common base names
- `Asphalt Mixture` - All asphalt mix types
- `Asphalt Cement` - Liquid asphalt/binder
- `Aggregate` - Stone, gravel, sand
- `Concrete` - Ready-mix concrete types

## Why this is better than filtering by hierarchy
The `--hierarchy-like` approach on material-types list may not capture all materials in a category. Using `material_type_fully_qualified_name_base` in the summary filter is:
- **Simpler**: No need to collect IDs first
- **More complete**: Captures all materials in the category
- **More maintainable**: New materials added to the category are automatically included
