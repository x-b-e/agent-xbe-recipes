---
title: Analyzing lineup dispatch creators by project manager
when: When you need to find out who created lineup dispatches for job production plans managed by a specific project manager, or when you need to trace dispatch creation patterns back to a PM
---

# Analyzing lineup dispatch creators by project manager

## Problem
You need to find out who created lineup dispatches for job production plans managed by a specific project manager.

## Solution

This is a multi-step process that requires:
1. Finding job production plans by project manager
2. Extracting lineup IDs from those plans
3. Getting all lineup dispatches for a time period
4. Filtering dispatches by the lineup IDs
5. Grouping by creator and counting

### Step 1: Find job production plans by project manager

```bash
xbe view job-production-plans list \
  --project-manager "<project-manager-name>" \
  --fields lineup,project-manager \
  --json \
  --limit 1000 > /tmp/jpp_plans.json
```

### Step 2: Extract lineup IDs into an array

```bash
jq '[.["lineup-id"]] | unique | sort' /tmp/jpp_plans.json > /tmp/lineup_ids_array.json
```

### Step 3: Get all lineup dispatches for the time period

Since there may be many dispatches (potentially 20,000+), you need to paginate:

```bash
# Get first batch
for offset in 0 1000 2000 3000 4000 5000; do 
  xbe view lineup-dispatches list \
    --created-at-min <start-date> \
    --created-at-max <end-date> \
    --fields lineup,created-by \
    --json \
    --limit 1000 \
    --offset $offset 2>&1
done | jq -s 'add' > /tmp/all_dispatches.json

# Check if more exist
xbe view lineup-dispatches list \
  --created-at-min <start-date> \
  --created-at-max <end-date> \
  --fields lineup,created-by \
  --json \
  --limit 1000 \
  --offset 6000 | jq 'length'

# If result is 1000, continue paginating
```

### Step 4: Filter dispatches by lineup IDs and count by creator

```bash
jq --slurpfile lineups /tmp/lineup_ids_array.json '
  [.[] | select(.["lineup-id"] as $lid | $lineups[0] | contains([$lid]))] |
  group_by(.["created-by"]) |
  map({creator: .[0]["created-by"], count: length}) |
  sort_by(-.count)
' /tmp/all_dispatches.json
```

### Step 5: If you have multiple batches, combine them

```bash
# Combine multiple batch files
cat /tmp/batch1_dispatches.json /tmp/batch2_dispatches.json | \
  jq -s 'add' > /tmp/combined_dispatches.json

# Then filter and count as in Step 4
```

## Tips

- **Pagination**: lineup-dispatches can have 20,000+ records for a month. Always check if you've retrieved all data by requesting one more batch beyond your last offset.

- **Efficient filtering**: Filter by lineup IDs using `jq --slurpfile` to avoid loading the lineup array multiple times.

- **Combining batches**: If you fetch data in multiple batches, you can either:
  - Filter each batch individually and combine the counts later
  - Combine all raw data first, then filter once (better if you have enough memory)

- **Memory considerations**: If you have 20,000+ records, processing them in batches of 5,000-10,000 and aggregating the results is more memory-efficient.

## Example output

```json
[
  {
    "creator": "<creator-name-1>",
    "count": 95
  },
  {
    "creator": "<creator-name-2>",
    "count": 31
  },
  {
    "creator": "<creator-name-3>",
    "count": 7
  }
]
```
