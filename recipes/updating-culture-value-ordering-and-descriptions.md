---
title: Updating culture value ordering and descriptions
when: When you need to reorder existing culture values and update their descriptions without replacing the entire set, especially when implementing a structured alternating pattern between categories
---

# Updating culture value ordering and descriptions

## When to use
When you need to reorder existing culture values and update their descriptions without replacing the entire set, especially when implementing a structured alternating pattern between categories.

## Pattern

### 1. First, view the current culture values to understand what needs to change
```bash
xbe view culture-values list --limit 1000 --json | jq '.[] | select(.organization == "<organization-name>")' | jq -s 'sort_by(.sequence_position)'
```

### 2. Update each value's position and description individually
```bash
xbe do culture-values update <culture-value-id> --position <new-position> --description "<new-description>"
```

### 3. Verify the final result with a formatted view
```bash
xbe view culture-values list --limit 1000 --json | jq '.[] | select(.organization == "<organization-name>")' | jq -s 'sort_by(.sequence_position) | .[] | {position: .sequence_position, name, description}'
```

## Example workflow

Implementing an alternating DOING/BEING pattern:

```bash
# Move and update first value
xbe do culture-values update 253 --position 0 --description "DOING: Taking ownership and accountability for our actions and outcomes. Works in balance with BEING values (Gifted, Humble, Quality, Beautiful)."

# Move and update second value
xbe do culture-values update 257 --position 1 --description "BEING: Recognizing and honoring our unique talents and the gifts we bring. Works in balance with DOING values (Responsible, Brave, Fast, Metabolic)."

# Continue for remaining values...

# Verify the final ordering and descriptions
xbe view culture-values list --limit 1000 --json | jq '.[] | select(.organization == "XBE Office")' | jq -s 'sort_by(.sequence_position) | .[] | {position: .sequence_position, name, description}'
```

## Notes
- The value name itself remains unchanged (e.g., "Responsible") - only the position and description are updated
- Labels like "DOING:" and "BEING:" go in the description field, not the name
- Use `--position` to reorder values (0-indexed)
- The verification jq query provides a clean view of position, name, and description for review
