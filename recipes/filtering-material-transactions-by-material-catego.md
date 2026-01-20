---
title: Filtering material transactions by material category
when: When you need to aggregate material transactions across all variants of a material category (e.g., all asphalt mixtures, all aggregates) rather than individual material types
---

# Filtering material transactions by material category

## Problem
Material types in XBE often have hierarchical names like "Asphalt Mixture:HMA:SB35", "Asphalt Mixture:HMA:SB292", etc. To get totals across all variants of a category, you need to filter by the base category name.

## Solution
Use the `material_type_fully_qualified_name_base` filter in material transaction summaries.

## Steps

### 1. First, identify what material types exist for a customer/broker
```bash
xbe summarize material-transaction-summary create \
  --group-by material_type \
  --filter customer=<customer-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date>
```

This shows you the material type names and their hierarchical structure.

### 2. Extract the base category name
From the output, identify the base category (the part before the first colon).
Examples:
- "Asphalt Mixture:HMA:SB35" → base is "Asphalt Mixture"
- "Aggregate:Sand:Type1" → base is "Aggregate"
- "Millings" → base is "Millings"

### 3. Filter by the base category
```bash
xbe summarize material-transaction-summary create \
  --group-by "" \
  --filter customer=<customer-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --filter material_type_fully_qualified_name_base="<base-category-name>" \
  --json
```

Using `--group-by ""` gives you a single total across all material types in that category.

## Example
To get total asphalt mixture tonnage for a customer:
```bash
xbe summarize material-transaction-summary create \
  --group-by "" \
  --filter customer=<customer-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --filter material_type_fully_qualified_name_base="Asphalt Mixture" \
  --json
```

## Notes
- This filter works with the hierarchical material type naming convention
- Use exact matching - the base name is case-sensitive
- Common base categories include: "Asphalt Mixture", "Aggregate", "Millings", "Concrete"
- Combine with other filters (customer, date ranges, etc.) as needed
