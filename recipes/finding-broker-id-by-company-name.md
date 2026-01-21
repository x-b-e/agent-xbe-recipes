---
title: Finding broker ID by company name
when: When you need to find a broker's ID to use in filters or other commands
---

# Finding broker ID by company name

To find a broker's ID when you know their company name:

```bash
xbe view brokers list --q "<company-name>"
```

The `--q` flag performs a server-side search query across broker attributes.

## Example

```bash
# Search for a broker by name
xbe view brokers list --q "Quantix"
```

## For exact ID lookup

If you already know the broker ID and want to verify:

```bash
xbe view brokers show <broker-id>
```

## Notes

- Use `--q` for fuzzy search, not `--name` 
- The search is case-insensitive and matches partial strings
- Returns a list of all matching brokers
- Add `--json` flag for programmatic parsing
