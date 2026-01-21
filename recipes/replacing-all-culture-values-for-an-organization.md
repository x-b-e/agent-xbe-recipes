---
title: Replacing all culture values for an organization
when: When you need to completely replace an organization's culture values with a new set, maintaining specific ordering
---

# Replacing all culture values for an organization

This recipe covers the complete workflow for replacing all culture values for an organization with a new set.

## Step 1: List existing culture values

First, identify all current culture values for the organization:

```bash
xbe view culture-values list --limit 1000 --json | jq '.[] | select(.organization == "<organization-name>") | {id, name, position: .sequence_position}'
```

## Step 2: Delete existing culture values

Delete each existing culture value by ID:

```bash
xbe do culture-values delete <culture-value-id>
```

Repeat for each culture value that needs to be removed.

## Step 3: Create new culture values in order

Create new culture values one at a time. The CLI automatically assigns sequence_position based on creation order (0, 1, 2, etc.):

```bash
xbe do culture-values create --name "<value-name>" --description "<value-description>" --organization "<organization-name>"
```

**Important**: Create values in the order you want them to appear, as the first created will have position 0, second will have position 1, etc.

## Step 4: Verify the new values

Confirm the new values were created in the correct order:

```bash
xbe view culture-values list --limit 1000 --json | jq '.[] | select(.organization == "<organization-name>") | {id, name, position: .sequence_position}' | jq -s 'sort_by(.position)'
```

## Tips

- The sequence_position is automatically assigned and increments with each new value created
- Plan your value order before creating them, as you'll need to create them sequentially
- Use descriptive names and descriptions that capture the full meaning of each value
- Consider grouping related values together in the sequence (e.g., all "doing" values followed by all "being" values)
