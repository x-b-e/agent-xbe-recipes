---
title: Analyzing asphalt sales by filtering material mixture types
when: When you need to analyze asphalt sales data and need to identify the correct material types representing finished asphalt products (HMA - Hot Mix Asphalt)
---

# Analyzing asphalt sales by filtering material mixture types

## Problem

When querying for "asphalt sales", you need to understand that asphalt-related materials are organized into different categories:
- **Asphalt aggregates** - limestone or other stone coated with asphalt
- **Asphalt cement** - the binder material (PG grades)
- **Asphalt mixtures** - finished HMA (Hot Mix Asphalt) products

For sales analysis, you typically want **asphalt mixtures** which represent the finished product sold to customers.

## Solution

### Step 1: Find all asphalt mixture material types

Use the `hierarchy-like` filter to find all material types under the "Asphalt Mixture" category:

```bash
xbe view material-types list \
  --broker <broker-id> \
  --hierarchy-like "Asphalt Mixture" \
  --json | jq -r 'map(.id) | unique | join(",")'
```

This returns a comma-separated list of material type IDs, for example:
```
10597,10598,10599,10602,10634,10637,11357,...
```

### Step 2: Query material transactions filtering by those types

Use the material type IDs to filter material transaction summaries:

```bash
xbe summarize material-transaction-summary create \
  --group-by "" \
  --filter broker=<broker-id> \
  --filter year=<year> \
  --filter month=<month> \
  --filter "material_type=<comma-separated-material-type-ids>" \
  --json
```

### Step 3: Query multiple months with a loop

For monthly breakdowns over a full year:

```bash
asphalt_types="<comma-separated-material-type-ids>"

for month in {1..12}; do
  result=$(xbe summarize material-transaction-summary create \
    --group-by "" \
    --filter broker=<broker-id> \
    --filter year=<year> \
    --filter month=$month \
    --filter "material_type=${asphalt_types}" \
    --json 2>&1)
  
  if echo "$result" | grep -q "tons_sum"; then
    tons=$(echo "$result" | jq -r '.rows[0].tons_sum // 0')
    echo "Month $month: ${tons} tons"
  fi
done
```

## Why month-by-month queries?

Querying a full year of asphalt data at once may time out (503 errors) due to the high transaction volume. Querying month-by-month is more reliable.

## Key insights

- **Asphalt Mixture** in the material hierarchy represents finished HMA products (what customers buy)
- **Asphalt Cement** represents the binder material (typically measured in tons but used as an ingredient)
- **Asphalt aggregates** are stone coated with asphalt (also typically ingredients)
- Use `hierarchy-like` to find all material types within a category rather than searching by partial name matches
