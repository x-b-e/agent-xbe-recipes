---
title: Grouping and counting action items by kind
when: When you need to see a breakdown of action items by their kind (feature, bug_fix, integration, training, etc.) to understand the distribution of work
---

# Grouping and counting action items by kind

## Problem
You need to understand how action items are distributed across different kinds (feature, bug_fix, integration, training, change_management, etc.).

## Solution
Use `--json` output with `jq` to group and count:

```bash
xbe view action-items list --status <status> --json | jq 'group_by(.kind) | map({kind: .[0].kind, count: length}) | sort_by(.count) | reverse'
```

This will output:
```json
[
  {
    "kind": "feature",
    "count": 29
  },
  {
    "kind": "integration",
    "count": 32
  }
]
```

## Handling large result sets
If you have more than 1000 items, you'll need to paginate. Save pages to temporary files and combine them:

```bash
# Fetch all pages
xbe view action-items list --status <status> --json > /tmp/action_items_1.json
xbe view action-items list --status <status> --offset 1000 --json > /tmp/action_items_2.json

# Combine and group
cat /tmp/action_items_1.json /tmp/action_items_2.json | jq -sr 'add | group_by(.kind) | map({kind: .[0].kind, count: length}) | sort_by(.count) | reverse'
```

## Notes
- The `-s` flag to `jq` reads the entire input as an array
- The `-r` flag produces raw output (no quotes around strings)
- `add` combines multiple arrays into one
- Some action items may have empty `kind` field
- You can group by other fields like `responsible_org_name`, `responsible_person_name`, or `status`
