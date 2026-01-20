---
title: Query broker material transactions by date range
when: user asks for hauling data, transaction data, or lane information for a specific broker over a time period
---

# Query Broker Material Transactions by Date Range

## When to use
Use when the user asks for hauling data, transaction data, or lane information for a specific broker over a time period.

## Steps

1. **Find the broker ID** by company name:
```bash
xbe view brokers list --company-name "<partial-name>" --json
```
This returns broker records with `id` and `company_name` fields.

2. **Query material transactions** for that broker:
```bash
xbe view material-transactions list --broker-id <broker-id> --start-date <YYYY-MM-DD> --end-date <YYYY-MM-DD> --json
```

3. **Handle pagination** if needed:
- Check if results are truncated (response will indicate if more data exists)
- Use `--limit` and `--offset` to paginate through large datasets
- For large datasets, save to a file and process incrementally

## Key fields in material-transactions
- `origin`: Origin location name
- `destination`: Destination location name  
- `trucker`: Trucking company name
- `quantity`: String value of tons (may be empty/null)
- `transaction_date`: Date in YYYY-MM-DD format

## Tips
- Always use `--json` for programmatic processing
- Date ranges filter the available data (may not have full year of history)
- Quantity fields are strings and may need filtering for empty/null values
- Use `jq` for complex analysis and aggregation
