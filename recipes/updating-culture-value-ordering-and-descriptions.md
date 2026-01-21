---
title: Updating culture value ordering and descriptions
when: When you need to reorder existing culture values and update their descriptions without replacing the entire set, especially when implementing a structured alternating pattern between categories
---

# Updating culture value ordering and descriptions

When you need to reorder existing culture values and update their descriptions (rather than replacing the entire set), use the `xbe do culture-values update` command with the `--position` and `--description` flags.

## Understanding the approach

- Each culture value has an ID that persists across updates
- You can update position and description independently or together
- Position updates reorder values (0-indexed)
- Description updates replace the entire description text

## Step-by-step workflow

1. **First, view the current state** to understand what needs to change:
```bash
xbe view culture-values list --limit 1000 --json | jq '.[] | select(.organization == "<org-name>")' | jq -s 'sort_by(.sequence_position) | .[] | {id, position: .sequence_position, name, description}'
```

2. **Update each value's position and description**:
```bash
xbe do culture-values update <value-id> --position <new-position> --description "<new-description>"
```

3. **Verify the final result**:
```bash
xbe view culture-values list --limit 1000 --json | jq '.[] | select(.organization == "<org-name>")' | jq -s 'sort_by(.sequence_position) | .[] | {position: .sequence_position, name, description}'
```

## Example: Implementing an alternating DOING/BEING pattern

If you need to reorder values to alternate between two categories:

```bash
# Move first BEING value to position 1
xbe do culture-values update <value-id> --position 1 --description "BEING: <description text>"

# Move first DOING value to position 0
xbe do culture-values update <value-id> --position 0 --description "DOING: <description text>"

# Continue alternating pattern...
```

## Tips

- Update positions sequentially to avoid conflicts
- Include category labels at the start of descriptions for clarity
- Consider explaining the relationship between categories in descriptions
- Use jq to filter and format output for verification
