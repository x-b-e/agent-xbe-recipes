---
title: Filtering memberships by user ID
when: When you need to filter memberships to show only those belonging to a specific user
---

# Filtering memberships by user ID

The `xbe view memberships list` command does not support a `--user-id` flag for filtering. To filter memberships by user, you need to use JSON output and `jq`.

## Example

```bash
xbe view memberships list --json | jq '.[] | select(.user_id == "<user-id>")'
```

This will return only memberships where the `user_id` field matches the specified ID.

## Getting the current user's memberships

First, get the current user's ID:

```bash
xbe auth whoami
```

Then filter memberships using that ID:

```bash
xbe view memberships list --json | jq '.[] | select(.user_id == "<user-id>")'
```

## Understanding empty results

If the filtered output is empty, the user has no memberships. This is normal for admin accounts, which have system-wide access without needing specific organizational memberships.
