---
title: Finding trucker ID for filtering
when: When you need to filter lane summaries or other data by trucker, especially when the trucker name contains special characters
---

## Problem

Filtering by trucker name directly in commands like `lane-summary create` can fail due to shell escaping issues, especially with names containing special characters like `&`.

## Solution

First look up the trucker ID, then use the numeric ID in your filters:

```bash
# Step 1: Find the trucker ID
xbe view truckers list --name "<partial-name>"

# Example output:
# ID    NAME              BROKER
# 1234  Acme Trucking     Example Broker

# Step 2: Use the ID in your filter
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter trucker=<trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by origin,destination
```

## Why This Works

- The `truckers list` command handles name matching properly with the `--name` flag
- Using numeric IDs avoids shell escaping problems
- Numeric IDs are more precise than partial name matches
