---
title: Listing user memberships
when: When you need to see what organizations a user belongs to and their role in each organization
---

# Listing user memberships

To view all memberships for the current user:

```bash
xbe view memberships list
```

This displays a table with:
- **ID**: Membership ID
- **USER**: The user's name
- **TYPE**: Organization type (e.g., Trucker, Broker)
- **NAME**: Organization name
- **KIND**: User's role in the organization (e.g., manager, operations)

## Understanding membership roles

Common role types:
- `manager`: Full management access to the organization
- `operations`: Operational access (typically more limited than manager)

## Notes

- A user can have multiple memberships across different organizations
- A user can have multiple roles within the same organization (e.g., both manager and operations roles)
- The command shows memberships for the currently authenticated user
