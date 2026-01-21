---
title: Calculating managed transport order percentages
when: When you need to find what percentage of transport orders are managed vs unmanaged for a broker
---

To calculate the percentage of managed transport orders, use the `transport-order-efficiency-summary` command grouped by `is_managed`. Note that the general `transport-summary` command does NOT support grouping by managed status.

## Basic Usage

```bash
xbe summarize transport-order-efficiency-summary create \
  --group-by is_managed \
  --filter broker=<broker-id> \
  --filter ordered_date_min=<start-date> \
  --filter ordered_date_max=<end-date>
```

## Example Output

```
is_managed  transport_order_count  ordered_miles_sum  routed_miles_sum  deviated_miles_sum
false       768                    133768.36          26657.27          126.02
true        45                     6989.01            13853.63          43.86
```

## Calculating the Percentage

From the output above:
- Total orders: 768 + 45 = 813
- Managed percentage: (45 / 813) × 100 = 5.5%
- Unmanaged percentage: (768 / 813) × 100 = 94.5%

## Important Notes

- Use `ordered_date` filters to match when orders were created
- The `transport-summary` command does NOT support filtering or grouping by managed status
- The `transport-orders list` command has an `--is-managed` filter but doesn't provide aggregate counts
- Different date filters (ordered_date vs pickup_date vs delivery_date) may yield different counts
