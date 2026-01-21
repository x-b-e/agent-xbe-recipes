---
title: Updating culture value ordering and descriptions
when: When you need to reorder existing culture values and update their descriptions without replacing the entire set, especially when implementing a structured alternating pattern between categories
---

# Updating culture value ordering and descriptions

When you need to reorder culture values and update their descriptions (without replacing the entire set), you can update both the position and description in a single command.

## Single command approach

Update both position and description in one command:

```bash
xbe do culture-values update <culture-value-id> --position <position> --description "<new-description>"
```

## Example: Implementing an alternating DOING/BEING pattern

When restructuring values to alternate between categories:

```bash
# Move first value to position 0 and update description
xbe do culture-values update <value-id-1> --position 0 --description "DOING: <description text>"

# Move second value to position 1 and update description  
xbe do culture-values update <value-id-2> --position 1 --description "BEING: <description text>"

# Move third value to position 2 and update description
xbe do culture-values update <value-id-3> --position 2 --description "DOING: <description text>"

# Continue pattern...
```

## Workflow for reorganizing culture values

1. First, list all current values to get their IDs:
```bash
xbe view culture-values list --limit 1000 --json | jq '.[] | select(.organization == "<organization-name>")'
```

2. Update each value with new position and description in sequence

3. Verify the final result:
```bash
xbe view culture-values list --limit 1000 --json | jq '.[] | select(.organization == "<organization-name>")' | jq -s 'sort_by(.sequence_position) | .[] | {position: .sequence_position, name, description}'
```

## Note about value names

The `name` field of the value remains unchanged - only the `description` and `sequence_position` are updated. If you need to change the actual value name, you would need to replace the entire culture value set using the replacement approach instead.
