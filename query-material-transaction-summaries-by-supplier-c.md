---
title: Query material transaction summaries by supplier/customer and date range
when: When you need to aggregate material transaction data (tons, counts) filtered by customer, supplier, material type, or date range
---

# Query Material Transaction Summaries by Supplier/Customer and Date Range

## Overview
Use `xbe do material-transaction-summary create` to get aggregated material transaction data filtered by supplier, customer, material type, or date range.

## Finding Supplier IDs Efficiently

Instead of listing all suppliers and filtering with jq, use the `--name` flag:

```bash
# Efficient: Use the --name filter
xbe view material-suppliers list --name "Supplier Name" --json

# Less efficient: List all and filter with jq
xbe view material-suppliers list --json | jq '.[] | select(.name | contains("Supplier Name"))'
```

**Note:** There may be multiple supplier entries with the same name but different brokers. Review the results to select the correct ID.

## Querying Material Transactions

### Basic Pattern
```bash
xbe do material-transaction-summary create \
  --group-by material_type \
  --filter material_supplier=<SUPPLIER_ID> \
  --filter date_min=YYYY-MM-DD \
  --filter date_max=YYYY-MM-DD \
  --json
```

### Filtering for Specific Material Types

Use jq to filter and sum specific material types (e.g., asphalt):

```bash
xbe do material-transaction-summary create \
  --group-by material_type \
  --filter material_supplier=129 \
  --filter date_min=2023-12-01 \
  --filter date_max=2023-12-31 \
  --json | jq '[.rows[] | select(.material_type_name and (.material_type_name | startswith("Asphalt Mixture")))] | map(.tons_sum) | add'
```

### Common Filters
- `material_supplier=<ID>` - Filter by supplier
- `material_customer=<ID>` - Filter by customer
- `material_type=<ID>` - Filter by material type
- `date_min=YYYY-MM-DD` - Start date
- `date_max=YYYY-MM-DD` - End date

### Group By Options
- `material_type` - Group by material type
- `date` - Group by date
- `supplier` - Group by supplier
- `customer` - Group by customer

## Tips
- Always use `--json` for programmatic processing
- Combine multiple filters with repeated `--filter` flags
- Use jq to filter rows by material name patterns and sum tons
- When filtering material names in jq, use `.material_type_name and (...)` to handle null values safely
