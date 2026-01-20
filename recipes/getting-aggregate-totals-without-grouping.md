---
title: Getting aggregate totals without grouping
when: When you need overall totals/aggregates from a lane summary without breaking down by any dimensions
---

To get aggregate totals (overall sum, median, etc.) without any grouping, pass an empty string to `--group-by`:

```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by "" \
  --metrics tons_sum
```

This returns a single row with the total across all dimensions:

```
tons_sum
1234567.89
```

This is different from omitting `--group-by` entirely, which would use a default grouping (typically origin,destination).
