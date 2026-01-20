---
title: Filtering memberships by user
when: When you need to see memberships for a specific user
---

# Filtering memberships by user

To filter memberships by user ID, use the `--user` flag:

```bash
xbe view memberships list --user <user-id>
```

Alternatively, if the flag approach doesn't work, filter using jq:

```bash
xbe view memberships list --json | jq '.[] | select(.user_id == "<user-id>")'
```

This is useful when you want to see all organizations a specific user belongs to, rather than all memberships across the system.

## Example output

```
ID  USER          TYPE    NAME              KIND
1   Sean Devine   Broker  XBE Demo          operations
2   Sean Devine   Broker  XBE Office        operations
4   Sean Devine   Broker  Quantix           manager
```
