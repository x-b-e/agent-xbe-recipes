---
title: Calculating trucker lane diversity with jq post-processing
when: When you need to calculate derived metrics from lane summary data, such as counting unique lanes per trucker or calculating concentration ratios
---

Lane summaries return raw data, but you can use `jq` to calculate additional metrics:

```bash
# Count unique lanes per trucker
xbe summarize lane-summary create \
  --group-by trucker,origin,destination \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --json | jq -r '.values[] | [.trucker, .origin, .destination] | @tsv' | \
  awk '{print $1}' | sort | uniq -c

# Calculate volume concentration (top 3 lanes as % of total)
xbe summarize lane-summary create \
  --group-by trucker,origin,destination \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --json | jq '
    .values 
    | group_by(.trucker) 
    | map({
        trucker: .[0].trucker,
        total_tons: (map(.tons_sum) | add),
        top3_tons: (sort_by(-.tons_sum) | .[0:3] | map(.tons_sum) | add),
        concentration_pct: ((sort_by(-.tons_sum) | .[0:3] | map(.tons_sum) | add) / (map(.tons_sum) | add) * 100)
      })
  '
```

This pattern is useful for leverage analysis, risk assessment, and understanding trucker dependencies.
