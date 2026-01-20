---
title: Find broker by company name
when: When you need to find a broker's ID by searching for their company name
---

To find a broker by company name, use the `--company-name` filter which supports partial matching:

```bash
xbe view brokers list --company-name "<search-term>" --json
```

Example:
```bash
xbe view brokers list --company-name "Geiger" --json
```

This returns:
```json
[
  {
    "id": "86",
    "company_name": "Geiger Ready-Mix"
  }
]
```

The broker ID can then be used with other commands like `--filter broker=<id>`.
