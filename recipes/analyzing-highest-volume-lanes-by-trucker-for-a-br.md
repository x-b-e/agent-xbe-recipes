---
title: Analyzing highest volume lanes by trucker for a broker
when: When you need to identify the most active shipping lanes for a broker, broken down by which trucking companies handled the volume
---

This workflow shows how to get the top lanes by tonnage for a specific broker, grouped by origin, destination, and trucker.

## Step 1: Find the broker ID

First, look up the broker's ID by company name:

```bash
xbe view brokers list --company-name "<broker-name>"
```

## Step 2: Generate lane summary grouped by trucker

Use the lane summary command with these key parameters:
- `--filter broker=<id>` to filter to the specific broker
- `--filter date_min` and `--filter date_max` for the time range
- `--group-by origin,destination,trucker` to break down by lane and trucker
- `--metrics tons_sum` to show total tonnage
- `--sort tons_sum:desc` to show highest volume first
- `--limit <n>` to control how many results

```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=2025-01-20 \
  --filter date_max=2026-01-20 \
  --group-by origin,destination,trucker \
  --metrics tons_sum \
  --sort tons_sum:desc \
  --limit 50
```

## Step 3 (Optional): Get total tonnage for context

To understand what percentage the top lanes represent, get the total:

```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=2025-01-20 \
  --filter date_max=2026-01-20 \
  --group-by "" \
  --metrics tons_sum
```

Note: Empty `--group-by ""` gives you the overall total without any grouping.
