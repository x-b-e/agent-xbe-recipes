---
title: Updating membership roles
when: When you need to change a user's role within an organization (e.g., from operations to manager or vice versa)
---

# Updating membership roles

## Context
Memberships define a user's relationship with an organization. The `kind` field determines whether they have an operations role or a manager role, which affects their permissions within the organization.

## Prerequisites
1. Know the membership ID you want to update
   - Use `xbe view memberships list --user <user-id>` to find membership IDs
2. Understand the role kinds:
   - `operations`: operational role with limited permissions
   - `manager`: management role with elevated permissions (e.g., can see rates as manager)

## Steps

### 1. Identify the membership to update
```bash
xbe view memberships list --user <user-id>
```

This will show all memberships for the user with their current role kind.

### 2. Update the membership role
```bash
xbe do memberships update <membership-id> --kind <new-role-kind>
```

Examples:
```bash
# Change from operations to manager
xbe do memberships update <membership-id> --kind manager

# Change from manager to operations
xbe do memberships update <membership-id> --kind operations
```

### 3. Verify the update
The command will return the updated membership details showing:
- The new role kind
- Updated permissions based on the new role
- Admin status (unchanged unless explicitly updated)

## Notes
- Changing to `manager` role typically grants permissions like "Can See Rates As Manager"
- Changing to `operations` role typically removes manager-level permissions
- Admin status is separate from role kind and must be updated independently if needed
- The update is immediate and affects the user's access to the organization
