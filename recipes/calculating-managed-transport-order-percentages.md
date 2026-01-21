---
title: Calculating managed transport order percentages
when: When you need to calculate the percentage of managed vs unmanaged transport orders for a broker
---

# Calculating managed transport order percentages

## Problem

The `xbe summarize transport-summary` command provides status breakdowns (active, at_risk, lifecycle states, etc.) but does NOT include a breakdown of managed vs unmanaged orders.

## Solution

Use `xbe view transport-orders list` with JSON output and `jq` to filter and count:

```bash
# Get managed order count
xbe view transport-orders list --broker <broker-id> --start-on <date> --end-on <date> --json --limit 10000 | jq '[.[] | select(.managed == true)] | length'

# Get total order count
xbe view transport-orders list --broker <broker-id> --start-on <date> --end-on <date> --json --limit 10000 | jq 'length'

# Calculate percentage with a single command
xbe view transport-orders list --broker <broker-id> --start-on <date> --end-on <date> --json --limit 10000 | jq '(([.[] | select(.managed == true)] | length) / length * 100)'
```

## Notes

- The `--is-managed` flag exists for filtering, but doesn't help with counting both categories
- Use `--limit 10000` to ensure you get all orders (default limit may be lower)
- The transport-summary command is useful for status breakdowns but not for managed/unmanaged analysis
