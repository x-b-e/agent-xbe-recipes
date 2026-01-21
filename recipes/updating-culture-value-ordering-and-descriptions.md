---
title: Updating culture value ordering and descriptions
when: When you need to reorder existing culture values and update their descriptions without replacing the entire set, especially when implementing a structured alternating pattern between categories (like DOING/BEING, yin/yang) where each value references its complementary values
---

# Updating culture value ordering and descriptions

## Context
When an organization restructures their culture values to follow an alternating pattern between complementary categories (e.g., DOING values and BEING values), you need to:
1. Reorder values to alternate between categories
2. Update descriptions to include category labels
3. Include cross-references showing how values work in balance

## Approach
Update each value individually using `xbe do culture-values update`, specifying both the new position and updated description.

## Example: Implementing DOING/BEING alternating pattern

First, list current values to see what needs updating:
```bash
xbe view culture-values list --limit 1000 --json | \
  jq '.[] | select(.organization == "<organization-name>")' | \
  jq -s 'sort_by(.sequence_position) | .[] | {id, position: .sequence_position, name}'
```

Then update each value with new position and category-labeled description:

```bash
# Update first DOING value (position 0)
xbe do culture-values update <value-id> \
  --position 0 \
  --description "DOING: Taking ownership and accountability for our actions and outcomes. Works in balance with BEING values (Gifted, Humble, Quality, Beautiful)."

# Update first BEING value (position 1)
xbe do culture-values update <value-id> \
  --position 1 \
  --description "BEING: Recognizing and honoring our unique talents and the gifts we bring. Works in balance with DOING values (Responsible, Brave, Fast, Metabolic)."

# Continue alternating pattern...
xbe do culture-values update <value-id> \
  --position 2 \
  --description "DOING: Having the courage to take risks, face challenges, and speak truth. Works in balance with BEING values (Gifted, Humble, Quality, Beautiful)."

# And so on for remaining values
```

## Verification
After updates, verify the final ordering and descriptions:

```bash
xbe view culture-values list --limit 1000 --json | \
  jq '.[] | select(.organization == "<organization-name>")' | \
  jq -s 'sort_by(.sequence_position) | .[] | {position: .sequence_position, name, description}'
```

## Key Points
- The value **name** remains unchanged (e.g., "Responsible", "Gifted")
- Only the **description** includes the category label (e.g., "DOING:", "BEING:")
- Each description cross-references the complementary values by name
- Update values one at a time to maintain data integrity
- Use `--position` to reorder values into the alternating pattern
- Verify the final result to ensure proper alternation and cross-references
