---
title: Calculating managed transport order percentages
when: When you need to find what percentage of transport orders are managed vs unmanaged for a broker
---

# Calculating managed transport order percentages

## Overview
Use `transport-order-efficiency-summary` grouped by `is_managed` to calculate what percentage of orders are managed vs unmanaged.

## Key Learning: Date Filter Selection
The efficiency summary supports two different date filters that return different results:
- `ordered_date` - filters by when the order was created
- `pickup_date` - filters by when the pickup is scheduled

For "today's orders" questions, **use `pickup_date`** as this represents the operational view of what's happening today.

## Basic Pattern
```bash
xbe summarize transport-order-efficiency-summary create \
  --group-by is_managed \
  --filter broker=<broker-id> \
  --filter pickup_date_min=<start-date> \
  --filter pickup_date_max=<end-date>
```

## Example: Today's managed percentage
```bash
# Get managed vs unmanaged breakdown for today's pickups
xbe summarize transport-order-efficiency-summary create \
  --group-by is_managed \
  --filter broker=<broker-id> \
  --filter pickup_date_min=<date> \
  --filter pickup_date_max=<date>
```

Output shows:
```
is_managed  transport_order_count  ordered_miles_sum  routed_miles_sum  deviated_miles_sum
true        398                    86268.39           168131.43         30581.18
false       367                    74872.54           120332.01         30607.07
```

Calculate percentage: 398 / (398 + 367) = 52% managed

## Alternative: Using ordered_date
If you need to see orders by when they were created (not when pickups are scheduled):
```bash
xbe summarize transport-order-efficiency-summary create \
  --group-by is_managed \
  --filter broker=<broker-id> \
  --filter ordered_date=<date>
```

This typically returns much fewer orders since it only includes orders created on that specific date.

## Available Date Filters
- `ordered_date` - specific date
- `ordered_date_min` / `ordered_date_max` - date range
- `pickup_date` - specific date  
- `pickup_date_min` / `pickup_date_max` - date range

## Related Commands
For status breakdowns without managed/unmanaged split, see `transport-summary`.
