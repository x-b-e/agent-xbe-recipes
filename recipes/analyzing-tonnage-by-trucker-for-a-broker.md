---
title: Analyzing tonnage by trucker for a broker
when: When you need to see total tonnage broken down by which truckers worked for a specific broker over a date range
---

# Analyzing tonnage by trucker for a broker

## Overview
To analyze tonnage by trucker for a broker, you need to:
1. Find the broker's ID
2. Use `lane-summary create` with trucker grouping and date filters

## Step 1: Find the broker ID
```bash
xbe view brokers list --company-name "<broker-name>"
```

This returns the broker ID you'll need for filtering.

## Step 2: Get tonnage by trucker
```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by trucker \
  --metrics tons_sum \
  --sort tons_sum:desc
```

### Key parameters:
- `--filter broker=<broker-id>`: Limits to a specific broker
- `--filter date_min=<start-date>` and `--filter date_max=<end-date>`: Define the time period (format: YYYY-MM-DD)
- `--group-by trucker`: Groups results by trucker to see each trucker's contribution
- `--metrics tons_sum`: Shows total tonnage
- `--sort tons_sum:desc`: Sorts by tonnage in descending order (highest first)

## Example output
The command returns a table showing each trucker and their total tonnage:
```
trucker_name                  tons_sum
Trucker A                     41170.43
Trucker B                     27116.06
Trucker C                     8836.72
...
```

## Related recipes
- See "Finding broker ID by company name" for more details on broker lookup
- See "Analyzing highest volume lanes by trucker" if you need to break down by specific lanes
- See "Creating ASCII bar charts from lane summaries" to visualize this data
