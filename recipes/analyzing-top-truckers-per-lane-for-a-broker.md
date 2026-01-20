---
title: Analyzing top truckers per lane for a broker
when: When you need to identify the top truckers operating on each of a broker's highest-volume lanes, creating a hierarchical view of lanes and their truckers
---

# Analyzing top truckers per lane for a broker

## Overview
This recipe shows how to perform hierarchical analysis: first identify top lanes for a broker, then find the top truckers on each of those lanes.

## Prerequisites
- Know the broker's ID (see recipe: finding-broker-id-by-company-name.md)
- Define your date range

## Step 1: Identify Top Lanes

First, get the top lanes for the broker:

```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by origin,destination \
  --sort tons_sum:desc \
  --limit <number-of-lanes>
```

This gives you the highest-volume origin-destination pairs.

## Step 2: Query Each Lane for Top Truckers

For each lane from Step 1, query again adding `trucker` to the group-by:

```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --filter origin_name="<origin-name>" \
  --filter destination_name="<destination-name>" \
  --group-by origin,destination,trucker \
  --sort tons_sum:desc \
  --limit 3
```

**Important**: Use exact origin and destination names from Step 1 results, including any special characters or spacing.

## Step 3: Run Queries in Parallel

For efficiency, run multiple lane queries in parallel rather than sequentially. This is especially important when analyzing 10+ lanes.

## Common Patterns Found

- **Single trucker dominance**: Many lanes are served by only one trucker (dedicated lane assignments)
- **Default unmanaged truckers**: These indicate self-performing contractors rather than third-party carriers
- **Competitive lanes**: Some lanes show multiple truckers competing, useful for identifying negotiation opportunities

## Example Output Interpretation

If a lane shows:
- 1 trucker with 100% of volume → Dedicated assignment, limited negotiation leverage
- 2-3 truckers with distributed volume → Competitive lane, better pricing opportunity
- "Default unmanaged trucker" → Self-performing work, not outsourced

## Tips

- Consider exporting results to a file for analysis (see recipe: creating-supplier-leverage-analysis-report-cards.md for deeper analysis)
- Some lanes may not appear in top 100 results if the broker has high lane diversity
- Total tonnage for truckers on a lane may not sum to the lane's total if there are more than 3 truckers

