---
title: Analyzing lanes by trucker
when: When you need to see all lanes (origin-destination pairs) operated by a specific trucker for a broker
---

## Overview

Use `lane-summary create` to analyze which lanes a specific trucker operates, showing tonnage, cycle counts, and transaction counts.

## Command Pattern

```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter trucker=<trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by origin,destination \
  --metrics tons_sum,cycle_count,material_transaction_count \
  --sort tons_sum:desc
```

## Key Points

- **Group by**: `origin,destination` shows each unique lane
- **Metrics**: Include `tons_sum` (volume), `cycle_count` (trips), and `material_transaction_count` (deliveries)
- **Sort**: `tons_sum:desc` shows highest volume lanes first
- **Date range**: Use date_min/date_max for the time period of interest

## Tips

- If you don't know the trucker ID, use `xbe view truckers list --name "<name>"` first
- To see all truckers across lanes, use `--group-by origin,destination,trucker` instead
- Remove `--sort` to see results in default order
