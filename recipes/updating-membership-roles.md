---
title: Updating membership roles
when: When you need to change a user's role within an organization (e.g., from manager to operations)
---

# Updating membership roles

## Problem
You need to change a user's role (kind) within an organization membership, such as changing from 'manager' to 'operations'.

## Solution
Use the `xbe do memberships update` command with the membership ID and the `--kind` flag:

```bash
xbe do memberships update <membership-id> --kind <role-kind>
```

## Valid role kinds
Common role kinds include:
- `operations`
- `manager`

## Example workflow

1. First, find the membership ID you want to update:
```bash
xbe view memberships list --user <user-id>
```

2. Update the membership role:
```bash
xbe do memberships update <membership-id> --kind operations
```

## Output
The command returns detailed information about the updated membership:
- ID and type
- User and organization details
- Updated role (kind and admin status)
- Permissions associated with the role
- Notification settings

## Notes
- Changing the role kind may automatically adjust associated permissions
- The output shows all permissions and notification settings for the updated membership

## Related recipes
- creating-memberships.md - Creating new memberships
- filtering-memberships-by-user-id.md - Finding memberships to update
- listing-user-memberships.md - Viewing user memberships
