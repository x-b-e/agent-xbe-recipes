---
title: Grouping action items by responsible party with counts
when: When you need to see how many action items are assigned to each person or organization, especially useful for workload distribution analysis
---

# Grouping action items by responsible party with counts

## Problem
You need to analyze action item distribution across people and organizations to understand workload.

## Solution

### Step 1: Fetch all action items with pagination
Since action items may exceed 1000, fetch multiple pages:

```bash
xbe view action-items list --json --page-size 1000 > /tmp/action_items_1.json
xbe view action-items list --json --page-size 1000 --page 2 > /tmp/action_items_2.json
```

### Step 2: Combine, filter, group, and count
Use jq to process the data:

```bash
cat /tmp/action_items_*.json | jq -sr '
  add 
  | map(select(.status == "<status>")) 
  | group_by(.responsible_person_name // .responsible_org_name // "Unassigned") 
  | map({
      responsible: (.[0].responsible_person_name // .[0].responsible_org_name // "Unassigned"), 
      count: length
    }) 
  | sort_by(.count) 
  | reverse
'
```

## Key points
- Use `responsible_person_name // responsible_org_name // "Unassigned"` to handle both person and organization assignments
- Filter by status (e.g., `in_progress`) before grouping to focus on relevant items
- The `-s` flag makes jq read all JSON objects into an array
- The `-r` flag provides raw output
- `add` combines multiple JSON arrays from different files
- Sort by count descending to see heaviest workloads first

## Common status values
- `in_progress`
- `ready_for_work`
- `editing`
- `completed`
- `abandoned`
