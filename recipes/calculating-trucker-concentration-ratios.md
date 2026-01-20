---
title: Calculating trucker concentration ratios
when: When you need to determine what percentage of a trucker's volume comes from their top lanes to assess their dependency on specific routes
---

# Calculating Trucker Concentration Ratios

Concentration ratio measures what percentage of a trucker's business comes from their top lanes. High concentration means the trucker is highly dependent on a few routes.

## Basic Concentration Ratio (Top 3 Lanes)

```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter trucker=<trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by origin,destination \
  --metrics tons_sum \
  --sort tons_sum:desc \
  --json | jq -r '
    .rows | 
    (map(.tons_sum | tonumber) | add) as $total | 
    (.[0:3] | map(.tons_sum | tonumber) | add) as $top3 | 
    "Top 3 concentration: \(($top3 / $total * 100 | floor))%"'
```

## Interpreting Concentration Ratios

- **High concentration (>60%)**: Trucker is very dependent on a few lanes with you - you have leverage
- **Medium concentration (30-60%)**: Balanced relationship
- **Low concentration (<30%)**: Trucker has diverse lane portfolio - they have more leverage

## Top 1 Lane Concentration

To see what percentage comes from their single biggest lane:

```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter trucker=<trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by origin,destination \
  --metrics tons_sum \
  --sort tons_sum:desc \
  --json | jq -r '
    .rows | 
    (map(.tons_sum | tonumber) | add) as $total | 
    (.[0].tons_sum | tonumber) as $top1 | 
    "Top 1 lane: \(($top1 / $total * 100 * 100 | floor) / 100)% (\($top1) of \($total) tons)"'
```

## Getting Multiple Concentration Metrics

Get top 1, top 3, and top 5 in one command:

```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter trucker=<trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by origin,destination \
  --metrics tons_sum \
  --sort tons_sum:desc \
  --json | jq -r '
    .rows | 
    (map(.tons_sum | tonumber) | add) as $total | 
    (.[0].tons_sum | tonumber) as $top1 | 
    (.[0:3] | map(.tons_sum | tonumber) | add) as $top3 | 
    (.[0:5] | map(.tons_sum | tonumber) | add) as $top5 | 
    "Top 1: \(($top1 / $total * 100 | floor))%\nTop 3: \(($top3 / $total * 100 | floor))%\nTop 5: \(($top5 / $total * 100 | floor))%"'
```

## Use in Leverage Analysis

Concentration ratio is a key indicator of negotiating power:

- A trucker with 80%+ concentration in top 3 lanes is highly vulnerable to you terminating those lanes
- A trucker with <25% concentration has many alternative lanes and less dependency on you
- Combined with total volume and lane diversity, this determines who has leverage in the relationship
