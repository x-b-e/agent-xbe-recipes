---
title: Calculating percentage of transport orders by managed status
when: When you need to calculate what percentage of transport orders are managed vs unmanaged for a broker on a specific date
---

# Calculating percentage of transport orders by managed status

To calculate the percentage of managed transport orders for a broker:

## Get count of managed orders

```bash
xbe view transport-orders list --broker <broker-id> --start-on <date> --end-on <date> --json | jq '[.[] | select(.managed == true)] | length'
```

## Get total count of orders

```bash
xbe view transport-orders list --broker <broker-id> --start-on <date> --end-on <date> --json | jq 'length'
```

## Calculate percentage

Divide managed count by total count and multiply by 100.

## Example workflow

```bash
# Count managed orders
managed=$(xbe view transport-orders list --broker <broker-id> --start-on <date> --end-on <date> --json | jq '[.[] | select(.managed == true)] | length')

# Count total orders
total=$(xbe view transport-orders list --broker <broker-id> --start-on <date> --end-on <date> --json | jq 'length')

# Calculate percentage
echo "scale=1; $managed * 100 / $total" | bc
```

## Notes

- Use the same date range for both queries to ensure accurate comparison
- The `--json` flag is required for jq processing
- This pattern works for any boolean filter (e.g., `active`, `unplanned`)
