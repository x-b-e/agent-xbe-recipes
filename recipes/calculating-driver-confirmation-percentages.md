---
title: Calculating driver confirmation percentages
when: When you need to calculate the percentage of PTP (Project Transport Plan) drivers who have confirmed their assignments for a broker over a date range
---

# Calculating driver confirmation percentages

## Overview
Driver confirmation percentage shows what portion of assigned PTP drivers have confirmed their transport assignments. This is calculated by dividing confirmed drivers by total assigned drivers.

## Step 1: Identify the confirmation metrics

First, check what confirmation-related metrics are available:

```bash
xbe summarize transport-order-efficiency-summary create --help | grep -i confirm
```

This will show you that `project_transport_plan_driver_confirmation_count` is available.

## Step 2: Query the data

Use the transport-order-efficiency-summary command with:
- Empty `--group-by` to get totals without grouping
- Date filters for your desired time period
- The three key metrics:

```bash
xbe summarize transport-order-efficiency-summary create \
  --group-by "" \
  --filter broker=<broker-id> \
  --filter pickup_date_min=<start-date> \
  --filter pickup_date_max=<end-date> \
  --metrics transport_order_count,project_transport_plan_driver_count,project_transport_plan_driver_confirmation_count
```

## Step 3: Calculate the percentage

From the output:
- `project_transport_plan_driver_count` = total PTP drivers assigned
- `project_transport_plan_driver_confirmation_count` = drivers who confirmed
- Driver Confirmation % = (confirmations / total drivers) × 100

## Example calculation

If the output shows:
```
transport_order_count  project_transport_plan_driver_count  project_transport_plan_driver_confirmation_count
3587                   2221.0                               229.0
```

Then: 229 / 2221 = 0.103 = **10.3% driver confirmation rate**

## Variations

### By day
Add `pickup_date` to `--group-by` to see daily trends:

```bash
xbe summarize transport-order-efficiency-summary create \
  --group-by pickup_date \
  --filter broker=<broker-id> \
  --filter pickup_date_min=<start-date> \
  --filter pickup_date_max=<end-date> \
  --metrics transport_order_count,project_transport_plan_driver_count,project_transport_plan_driver_confirmation_count
```

### By office
Add `project_office` to `--group-by` to break down by office:

```bash
xbe summarize transport-order-efficiency-summary create \
  --group-by project_office \
  --filter broker=<broker-id> \
  --filter pickup_date_min=<start-date> \
  --filter pickup_date_max=<end-date> \
  --metrics transport_order_count,project_transport_plan_driver_count,project_transport_plan_driver_confirmation_count
```
