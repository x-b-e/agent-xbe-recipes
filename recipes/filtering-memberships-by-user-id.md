---
title: Filtering memberships by user
when: When you need to see memberships for a specific user
---

# Filtering memberships by user

## Problem
You need to view only the memberships for a specific user rather than all memberships in the system.

## Solution
Use the `--user` flag with the memberships list command:

```bash
xbe view memberships list --user <user-id>
```

## Example workflow

1. First, identify the user ID (if needed):
```bash
xbe auth whoami
```

This shows your current user ID and details.

2. Then filter memberships by that user:
```bash
xbe view memberships list --user <user-id>
```

## Output
The command returns a table showing:
- Membership ID
- User name
- Organization type (Broker, Trucker, etc.)
- Organization name
- Role kind (manager, operations, etc.)

## Related recipes
- listing-user-memberships.md - General membership listing
- creating-memberships.md - Creating new memberships
