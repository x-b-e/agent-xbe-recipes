---
title: Use material-transaction-summary for aggregated data
when: User asks for totals, sums, or aggregated material data across time periods
---

# Use material-transaction-summary instead of listing individual transactions

When you need to calculate totals or aggregates (like total tons sold in a month), use:

```bash
xbe do material-transaction-summary create \
  --group-by material_type \
  --filter material_supplier=129 \
  --filter date_min=2023-12-01 \
  --filter date_max=2023-12-31 \
  --json
```

Instead of fetching thousands of individual transactions with `xbe view material-transactions list`.

## Key benefits:
- Much faster (one API call vs potentially thousands)
- Server-side aggregation
- Supports grouping by many dimensions (material_type, date, customer, etc.)
- Returns pre-calculated sums and counts

## Filtering asphalt materials:
Most asphalt products are named with pattern "Asphalt Mixture:HMA:*". Use jq to filter:

```bash
jq '[.rows[] | select(.material_type_name != null and (.material_type_name | startswith("Asphalt Mixture")))] | map(.tons_sum) | add'
```

## Common patterns:
- Group by material_type to see breakdown by product
- Group by date to see daily trends  
- Group by customer to see who bought what
- Use empty group-by (`--group-by ""`) for grand totals
