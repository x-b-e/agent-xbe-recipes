---
title: Analyzing lane concentration by trucker
when: When you need to see which truckers serve which specific lanes (origin-destination pairs) and their volume on each lane
---

To see volume broken down by both trucker and lane:

```bash
xbe summarize lane-summary create \
  --group-by trucker,origin,destination \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --json
```

This creates a detailed breakdown showing:
- Which truckers serve which lanes
- How much volume each trucker handles on each lane
- Lane specialization patterns (e.g., if a trucker only serves 2-3 lanes)

Useful for:
- Identifying lane dependencies (e.g., trucker X dominates lane Y)
- Understanding trucker specialization vs. diversification
- Supply chain risk analysis
