---
title: Updating membership roles
when: When you need to change a user's role within an organization (e.g., from operations to manager or vice versa)
---

# Updating membership roles

To update a user's role within an organization membership:

## Steps

1. First, find the membership ID you want to update:
```bash
xbe view memberships list --user <user-id>
```

2. Update the membership role using the membership ID:
```bash
xbe do memberships update <membership-id> --kind <operations|manager>
```

## Example

```bash
# List memberships for user 1 to find the membership ID
xbe view memberships list --user <user-id>

# Update membership from manager to operations
xbe do memberships update <membership-id> --kind operations

# Update membership from operations to manager
xbe do memberships update <membership-id> --kind manager
```

## Notes

- The `--kind` flag accepts two values: `operations` or `manager`
- You need the membership ID (not the user ID or organization ID) to perform the update
- The update command will show the full membership details after the change is applied
- Other membership properties (like admin status, title, permissions) remain unchanged unless explicitly specified
