---
title: Filtering memberships by user
when: When you need to see memberships for a specific user, or when a user asks for 'my memberships' and you need to identify who they are first
---

# Filtering memberships by user

## Overview
Use the `--user` flag with `xbe view memberships list` to filter memberships for a specific user.

## Finding the current user's ID
When a user asks for "my memberships" or "my personal memberships", first identify who they are:

```bash
xbe auth whoami
```

This returns the current authenticated user's information including their ID:
```
Logged in as <user-name>
  ID:    <user-id>
  Email: <email>
  Mobile: <phone>
  Admin: <yes|no>
```

## Filtering by user ID
Once you have the user ID, filter memberships:

```bash
xbe view memberships list --user <user-id>
```

## Example workflow
1. Get current user info: `xbe auth whoami`
2. Extract the user ID from output
3. Filter memberships: `xbe view memberships list --user <user-id>`

## Notes
- Without the `--user` flag, `xbe view memberships list` returns all memberships in the system
- The `--user` flag filters to show only memberships for the specified user
- Each membership shows the organization type (Broker, Trucker, etc.) and the user's role (manager, operations, etc.)
