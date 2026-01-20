---
title: Analyzing top truckers per lane for a broker
when: When you need to identify the top trucking companies operating on each of a broker's highest-volume lanes
---

# Analyzing top truckers per lane for a broker

## Overview
This recipe shows how to identify the top lanes for a broker, then drill down to find which trucking companies handle the most tonnage on each lane.

## Steps

### 1. Find the broker ID
First, identify the broker you're analyzing:

```bash
xbe view brokers list --company-name "<broker-name>"
```

### 2. Get the top lanes by tonnage
Query the top N lanes for the broker over your date range:

```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by origin,destination \
  --sort tons_sum:desc \
  --limit <number-of-lanes>
```

### 3. Get top truckers for each lane
For each lane identified in step 2, query the top truckers on that specific origin-destination pair:

```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --filter origin_name="<origin-name>" \
  --filter destination_name="<destination-name>" \
  --group-by origin,destination,trucker \
  --sort tons_sum:desc \
  --limit <number-of-truckers>
```

**Important**: Use exact origin and destination names from step 2, including quotes to handle spaces and special characters.

### 4. Filter out default unmanaged truckers
When analyzing the results, exclude entries labeled "Default unmanaged trucker" - these represent self-performing contractors (the customer's own trucks) rather than external trucking companies.

## Tips
- Run queries for multiple lanes in parallel to speed up analysis
- The `--limit` flag controls how many top truckers to return per lane
- Some lanes may only have one trucker or may be entirely self-performed
- Use `tons_sum` for sorting by total tonnage, or `cycle_count` for sorting by number of trips

## Example Output Interpretation
- If a lane shows only "Default unmanaged trucker", it's entirely self-performed
- If one trucker dominates with 90%+ of tonnage, that lane has no real competition
- Multiple truckers with significant tonnage indicates competitive sourcing on that lane
