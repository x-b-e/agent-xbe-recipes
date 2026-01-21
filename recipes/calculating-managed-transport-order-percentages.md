---
title: Calculating managed transport order percentages
when: When you need to calculate the percentage of managed vs unmanaged transport orders for a broker
---

## Overview

To calculate what percentage of transport orders are managed vs unmanaged for a broker, use the `transport-order-efficiency-summary` command with `--group-by is_managed`.

## Key Concepts

### Date Filters Matter

The efficiency summary supports two different date filters that produce different results:

- **ordered_date** - When the order was created
- **pickup_date** - When the pickup was scheduled

For questions about "today's orders" or "orders happening today", use `pickup_date` filters as this reflects operational activity. Use `ordered_date` filters when analyzing when orders were placed.

## Basic Command

```bash
xbe summarize transport-order-efficiency-summary create \
  --group-by is_managed \
  --filter broker=<broker-id> \
  --filter pickup_date_min=<start-date> \
  --filter pickup_date_max=<end-date>
```

## Example Output

```
is_managed  transport_order_count  ordered_miles_sum  routed_miles_sum  deviated_miles_sum
true        398                    86268.39           168131.43         30581.18
false       367                    74872.54           120332.01         30607.07
```

Calculate percentage:
- Managed: 398 / (398 + 367) = 52%
- Unmanaged: 367 / (398 + 367) = 48%

## Date Filter Examples

### For today's pickups:
```bash
xbe summarize transport-order-efficiency-summary create \
  --group-by is_managed \
  --filter broker=<broker-id> \
  --filter pickup_date_min=<today-date> \
  --filter pickup_date_max=<today-date>
```

### For orders created today:
```bash
xbe summarize transport-order-efficiency-summary create \
  --group-by is_managed \
  --filter broker=<broker-id> \
  --filter ordered_date_min=<today-date> \
  --filter ordered_date_max=<today-date>
```

## Available Group-by Attributes

The efficiency summary supports many group-by options if you need breakdowns:
- broker
- customer
- driver
- is_managed
- ordered_date
- pickup_date
- project_category
- project_division
- project_office
- transport_order
- transport_order_status
- was_unmanaged

## Available Filters

- broker
- customer
- driver
- is_managed (true/false)
- was_unmanaged (true/false)
- ordered_date / ordered_date_min / ordered_date_max
- pickup_date / pickup_date_min / pickup_date_max
- project_division
- project_office
- project_category
- transport_order
- transport_order_status

## Related Commands

- `xbe summarize transport-summary` - Status breakdowns but does NOT support filtering by managed status
- `xbe view transport-orders list` - Individual order listing with --is-managed filter
