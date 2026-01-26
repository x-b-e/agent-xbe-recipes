---
title: Filtering material transactions by material hierarchy
when: When you need to filter material transaction summaries by a broad material category (like 'Asphalt Mixture', 'Aggregate', 'Concrete') rather than individual material types, and you need to find all the specific material type IDs that belong to that category
---

# Filtering material transactions by material hierarchy

## Problem

You need to analyze material transactions for a broad category (e.g., all asphalt mixtures, all aggregates) but the `material-transaction-summary` requires specific material type IDs, not category names.

## Solution

Use a two-step process:

1. **Find all material type IDs under the category** using `--hierarchy-like`:
```bash
xbe view material-types list --broker <broker-id> --hierarchy-like "<category-name>" --json | jq -r 'map(.id) | unique | join(",")'
```

2. **Use those IDs in your material transaction summary filter**:
```bash
material_type_ids="<comma-separated-ids-from-step-1>"

xbe summarize material-transaction-summary create \
  --group-by "" \
  --filter broker=<broker-id> \
  --filter year=<year> \
  --filter "material_type=${material_type_ids}" \
  --json
```

## Example: Getting monthly asphalt sales

```bash
# Step 1: Get all asphalt mixture material type IDs
asphalt_types=$(xbe view material-types list --broker <broker-id> --hierarchy-like "Asphalt Mixture" --json | jq -r 'map(.id) | unique | join(",")')

# Step 2: Query monthly totals for all asphalt types
for month in {1..12}; do
  xbe summarize material-transaction-summary create \
    --group-by "" \
    --filter broker=<broker-id> \
    --filter year=<year> \
    --filter month=$month \
    --filter "material_type=${asphalt_types}" \
    --json | jq -r '.rows[0].tons_sum // 0'
done
```

## Key Points

- The `--hierarchy-like` flag searches for material types whose hierarchy path contains the search term
- Material hierarchies use colon-separated paths like `Asphalt Mixture:HMA:SB97`
- Using `jq` to extract unique IDs and join them with commas creates the right format for filtering
- This approach captures all variants under a category without manually listing each type

## Related Material Categories

Common hierarchy patterns to search for:
- `"Asphalt Mixture"` - All HMA, asphalt mixes
- `"Asphalt Cement"` - Asphalt binders
- `"Aggregate"` - All aggregates
- `"Concrete"` - Ready-mix concrete
- `"Sand"` - Sand products
- `"Stone"` - Stone products
