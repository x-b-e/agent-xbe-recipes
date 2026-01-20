---
title: Finding broker ID by company name
when: When you need to find a broker's ID to use in filters or other commands
---

Use the `xbe view brokers list` command with the `--company-name` flag to search for brokers by partial name match:

```bash
xbe view brokers list --company-name "<search-term>"
```

Example:
```bash
xbe view brokers list --company-name "Acme"
```

Output:
```
ID  COMPANY
123  Acme Corporation
```

The ID can then be used in other commands like lane summaries:
```bash
xbe summarize lane-summary create --filter broker=<broker-id> ...
```
