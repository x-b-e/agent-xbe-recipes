---
title: Filtering action items by organization
when: When you need to filter action items by the responsible organization (e.g., to see only internal XBE items or items assigned to a specific customer/broker)
---

# Filtering action items by organization

## Problem

You want to filter action items to show only those assigned to a specific organization (e.g., internal XBE items vs external customer/broker items).

## Solution

Use the `responsible_org_name` field to filter action items. This field is available in the action items data structure.

### Filter for XBE Office (internal) items

```bash
xbe view action-items list --status in_progress --json | \
  jq 'map(select(.responsible_org_name == "XBE Office"))'
```

### Filter for a specific customer/broker organization

```bash
xbe view action-items list --status in_progress --json | \
  jq 'map(select(.responsible_org_name == "<org-name>"))'
```

### Count items by organization

```bash
xbe view action-items list --status in_progress --json | \
  jq 'group_by(.responsible_org_name) | map({org: .[0].responsible_org_name, count: length})'
```

## Key fields for filtering

Action items have several organization-related fields:
- `responsible_org_name`: Name of the responsible organization
- `responsible_org_id`: ID of the responsible organization
- `responsible_org_type`: Type of organization (e.g., broker, customer)
- `responsible_person_name`: Name of the specific person responsible (if assigned)

## Notes

- When filtering for internal items, use `responsible_org_name == "XBE Office"`
- Some items may have a person assigned but still belong to an organization
- You can combine organization filters with other filters like status, kind, or priority
