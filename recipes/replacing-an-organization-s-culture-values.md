---
title: Replacing an organization's culture values
when: When you need to replace an organization's entire culture value system with a new set of values while maintaining proper ordering/sequencing
---

# Replacing an organization's culture values

## Context
Culture values in XBE have a `sequence_position` field that determines their display order. When replacing all culture values for an organization, you need to delete the old values and create new ones with the correct positions.

## Steps

### 1. View existing culture values
First, identify what culture values currently exist for the organization:

```bash
xbe view culture-values list --limit 1000 --json | jq '.[] | select(.organization == "<organization-name>") | {id, name, position: .sequence_position}' | jq -s 'sort_by(.position)'
```

This shows you the current values, their IDs, and their positions.

### 2. Delete existing culture values
Delete each existing culture value by ID:

```bash
xbe do culture-values delete <culture-value-id>
```

Repeat for each value you identified in step 1.

### 3. Create new culture values in order
Create the new culture values. The `xbe do culture-values create` command automatically assigns sequential positions based on creation order, so create them in the order you want them displayed:

```bash
xbe do culture-values create --name "<value-name>" --description "<description>" --organization "<organization-type>|<organization-id>"
```

For example, if you want positions 0-7, create 8 values in the desired order.

**Important:** The organization parameter uses the format `"<OrganizationType>|<id>"` (e.g., `"Broker|49"`).

### 4. Verify the new values
Confirm the new values were created with correct positions:

```bash
xbe view culture-values list --limit 1000 --json | jq '.[] | select(.organization == "<organization-name>") | {id, name, position: .sequence_position}' | jq -s 'sort_by(.position)'
```

## Tips
- Culture values are automatically assigned sequential positions (0, 1, 2, etc.) based on creation order
- If you need to reorder existing values, you'll need to delete and recreate them
- Consider the semantic meaning of the order - grouped vs alternating arrangements can convey different organizational philosophies
