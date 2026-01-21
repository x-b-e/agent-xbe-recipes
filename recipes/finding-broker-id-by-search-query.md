---
title: Finding broker ID by search query
when: When you need to find a broker's ID using their name or partial name match
---

# Finding broker ID by search query

To find a broker's ID when you know their name:

```bash
xbe view brokers list --q "<search-term>"
```

The `--q` flag performs a server-side search across broker attributes.

## Example

```bash
# Find Quantix broker
xbe view brokers list --q "Quantix"
```

## Notes

- Use `--q` for search queries, not `--name` (which doesn't exist for brokers)
- The search is case-insensitive and matches partial strings
- Returns a list of matching brokers with their IDs
- Add `--json` flag for programmatic parsing of results
