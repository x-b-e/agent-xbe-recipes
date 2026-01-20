---
title: Analyzing Lane Volumes by Trucker with XBE CLI
when: When you need to analyze transportation lanes, volumes, or tonnage data for a broker, grouped by various dimensions like origin, destination, and trucker
---

# Analyzing Lane Volumes by Trucker with XBE CLI

## Overview
Use `xbe summarize lane-summary create` to analyze transportation lane data with flexible grouping and filtering.

## Key Commands

### 1. Get help on available options
```bash
xbe summarize lane-summary create --help
```

### 2. Find the broker ID first
```bash
xbe view brokers list --json
```

### 3. Analyze lanes grouped by origin, destination, and trucker
```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by origin,destination,trucker \
  --metrics tons_sum,cycle_count,material_transaction_count \
  --sort tons_sum:desc \
  --limit 50
```

### 4. Get overall totals (no grouping)
```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by "" \
  --metrics tons_sum,cycle_count,material_transaction_count \
  --json
```

### 5. Get top lanes without trucker breakdown
```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by origin,destination \
  --metrics tons_sum,cycle_count,material_transaction_count \
  --sort tons_sum:desc \
  --limit 20
```

## Important Details

- **Date formats**: Use ISO format (YYYY-MM-DD) for date_min/date_max
- **Empty group-by**: Use `--group-by ""` to get totals without any grouping
- **Default grouping**: If you don't specify `--group-by`, it defaults to `origin,destination`
- **Default metrics**: Without `--metrics`, you get: cycle_count, material_transaction_count, cycle_minutes_median, calculated_travel_minutes_median, tons_sum
- **Sorting**: Use format `--sort <field>:desc` or `--sort <field>:asc`
- **JSON output**: Add `--json` flag for machine-readable output

## Common Grouping Dimensions
- `origin,destination` - lanes
- `origin,destination,trucker` - lanes by carrier
- `trucker` - by carrier only
- `material_type` - by material
- `customer` - by customer
- `date` or `month` or `year` - time-based analysis
- `""` - no grouping (totals only)

## Available Metrics
- `tons_sum` - total tonnage
- `cycle_count` - number of cycles
- `material_transaction_count` - number of transactions
- `cycle_minutes_median` - median cycle time
- `calculated_travel_minutes_median` - median travel time
- Many more available - see `--help` for full list
