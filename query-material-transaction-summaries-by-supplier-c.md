---
title: Query material transaction summaries by supplier/customer and date range
when: When you need to aggregate material transaction data (tons, counts) filtered by customer, supplier, material type, or date range
---

# Finding Material Transaction Summaries in XBE

## Step 1: Find the entity ID
First, look up the customer/supplier ID:
```bash
# For customers
xbe view customers list --json | jq '.[] | select(.name | contains("SearchTerm"))'

# For material suppliers
xbe view material-suppliers list --json | jq '.[] | select(.name | contains("SearchTerm"))'
```

## Step 2: Create a material transaction summary
Use the 'do' command (not 'view') to create summaries:
```bash
xbe do material-transaction-summary create \
  --group-by material_type \
  --filter customer=<ID> \
  --filter date_min=YYYY-MM-DD \
  --filter date_max=YYYY-MM-DD \
  --json
```

Alternatively, filter by supplier:
```bash
xbe do material-transaction-summary create \
  --group-by material_type \
  --filter supplier=<ID> \
  --filter date_min=YYYY-MM-DD \
  --filter date_max=YYYY-MM-DD \
  --json
```

## Step 3: Calculate totals with jq
To sum tons for specific material types:
```bash
# Sum all asphalt mixture tons
xbe do material-transaction-summary create ... --json | \
  jq '[.rows[] | select(.material_type_name and (.material_type_name | startswith("Asphalt Mixture")))] | map(.tons_sum) | add'
```

## Key learnings
- Use `xbe view` for listing/browsing resources
- Use `xbe do` for creating summaries and aggregations
- Filter syntax: `--filter key=value` (can use multiple filters)
- Common group-by options: material_type, customer, supplier, date
- The response has a `.rows[]` array with aggregated data
