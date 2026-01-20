---
title: Viewing action items (read-only access)
when: When you need to view or list action items from the XBE platform. Note: The CLI currently only supports read-only access to action-items - you cannot create, update, or change their status.
---

# Viewing Action Items (Read-Only Access)

## Overview
The xbe CLI provides read-only access to action items through the `view action-items` commands. Currently, there are no `do action-items` commands available, so you can only browse and view action items but cannot create, update, or change their status.

## Available Commands

### List action items
```bash
xbe view action-items list
```

This returns action items grouped by person/organization, showing:
- The responsible person or organization
- Action item ID
- Action item kind (feature, bug_fix, integration, training, change_management, etc.)
- Action item title

### View action item details
```bash
xbe view action-items show <action-item-id>
```

## Output Format
By default, the output is JSON with this structure:
```json
{
  "person": "<person-or-organization-name>",
  "items": [
    {
      "id": "<action-item-id>",
      "kind": "<kind>",
      "title": "<title>"
    }
  ]
}
```

## Filtering
You can filter action items by status using the `--status` flag:
```bash
xbe view action-items list --status in_progress
```

See the "Listing open action items" recipe for details on filtering by status.

See the "Filtering action items by organization" recipe for filtering by responsible party.

## Limitations
- **Read-only access**: You cannot create, update, delete, or change the status of action items through the CLI
- To take action on items, you must use the web interface or wait for `do action-items` commands to be added to the CLI
