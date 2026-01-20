---
title: Paginating through large result sets
when: When you need to retrieve more than 1000 items from any xbe CLI list command that returns paginated results
---

## Problem

The xbe CLI enforces a maximum page size of 1000 items. When querying resources that exceed this limit, you need to use pagination.

## Solution

Use the `--limit` and `--offset` parameters together to retrieve all items:

```bash
# Get the first 1000 items
xbe view <resource> list --limit 1000 --json > page1.json

# Get the next batch using offset
xbe view <resource> list --limit 1000 --offset 1000 --json > page2.json

# Continue with additional offsets if needed
xbe view <resource> list --limit 1000 --offset 2000 --json > page3.json
```

## Combining Results

Use `jq -s 'add'` to combine multiple paginated JSON files:

```bash
cat page1.json page2.json page3.json | jq -s 'add'
```

## Determining Total Count

To find out how many pages you need:

```bash
# Check if you hit the limit
xbe view <resource> list --limit 1000 --json | jq 'length'
# If this returns 1000, there may be more items

# Check the next page
xbe view <resource> list --limit 1000 --offset 1000 --json | jq 'length'
# If this returns 0, you've retrieved everything
# If non-zero, continue with additional offsets
```

## Example: Getting All Open Action Items

```bash
# Retrieve all open action items (1,448 total)
xbe view action-items list \
  --status editing,ready_for_work,in_progress,in_verification,on_hold \
  --limit 1000 --json > /tmp/action_items_1.json

xbe view action-items list \
  --status editing,ready_for_work,in_progress,in_verification,on_hold \
  --limit 1000 --offset 1000 --json > /tmp/action_items_2.json

# Combine and analyze
cat /tmp/action_items_1.json /tmp/action_items_2.json | jq -s 'add | length'
```

## Important Notes

- The maximum `--limit` value is 1000. Attempting to use a higher value will return a 400 error.
- Always check if there are more items by testing the next offset.
- This pattern works for any xbe CLI list command (action-items, newsletters, customers, etc.).
