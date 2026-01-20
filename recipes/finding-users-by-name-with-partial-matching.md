---
title: Finding users by name with partial matching
when: When you need to find a user's ID but don't know the exact name format stored in the system
---

# Finding users by name with partial matching

## The challenge

User names in XBE may be stored in various formats (e.g., "Patrick K Mulvany" vs "Pat Mulvaney"), and the search needs to match what's actually in the database.

## Search strategy

Try increasingly general searches:

1. **Start with full name**:
```bash
xbe view users list --name "<first-name> <last-name>" --json
```

2. **If no results, try last name only**:
```bash
xbe view users list --name "<last-name>" --json
```

3. **If still no results, try first name only**:
```bash
xbe view users list --name "<first-name>" --json
```

4. **If you get multiple results, filter by other fields**:
- Check the `email` field for domain matching
- Check the `mobile` field if you have it
- Check organizational context if you know which broker/company they work for

## Example

Searching for "Pat Mulvaney":

```bash
# Try full name - may return empty
xbe view users list --name "Pat Mulvaney" --json

# Try last name - may return empty  
xbe view users list --name "Mulvaney" --json

# Try first name - finds "Patrick K Mulvany"
xbe view users list --name "Pat" --json
```

## Tips

- The `--name` flag does partial matching, but only on what's stored
- Common name variations: nicknames (Pat/Patrick), middle initials, suffixes (Jr., Sr.)
- The search is case-insensitive
- If you get many results, combine with `| jq` to filter by other fields
