---
title: Finding customer material transaction summaries in xbe CLI
when: When you need to query material transactions for a specific customer and calculate totals by material type
---

# Finding and Analyzing Customer Material Transactions

## 1. Find the customer ID
Use the --name flag to search for customers:
```bash
xbe view customers list --name "Customer Name" --json
```

Note: The JSON output is a plain array, not wrapped in a data field:
```bash
# Correct:
xbe view customers list --json | jq '.[] | select(...)'

# Incorrect:
xbe view customers list --json | jq '.data[] | select(...)'
```

## 2. Get material transaction summaries
Use the material-transaction-summary command with filters:
```bash
xbe do material-transaction-summary create \
  --group-by material_type \
  --filter customer=<customer_id> \
  --filter date_min=YYYY-MM-DD \
  --filter date_max=YYYY-MM-DD \
  --json
```

## 3. Filter and sum specific material types
To calculate totals for specific material types (e.g., only asphalt mixtures):
```bash
xbe do material-transaction-summary create \
  --group-by material_type \
  --filter customer=<id> \
  --filter date_min=YYYY-MM-DD \
  --filter date_max=YYYY-MM-DD \
  --json | jq '[.rows[] | select(.material_type_name and (.material_type_name | startswith("Material Prefix")))] | map(.tons_sum) | add'
```

The jq filter:
- Selects rows where material_type_name starts with a specific prefix
- Maps to extract tons_sum values
- Adds them together for a total
