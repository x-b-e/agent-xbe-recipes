---
title: Search for broker by name
when: User wants to find a specific broker or needs a broker ID
---

# Search for broker by name

Use the `--company-name` flag to search for brokers by partial name match:

```bash
xbe view brokers list --company-name "<search-term>" --json
```

The search performs a partial match, so you don't need the exact company name.

Example:
```bash
# Find broker with "Geiger" in the name
xbe view brokers list --company-name "Geiger" --json

# Extract just the ID
xbe view brokers list --company-name "Geiger" --json | jq -r '.[0].id'
```

This is more efficient than listing all brokers and using grep.
