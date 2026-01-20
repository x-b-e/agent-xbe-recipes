---
title: Listing user memberships
when: When you need to see what organizations a user belongs to and their role in each organization
---

# Listing user memberships

## Problem
You need to see what organizations a specific user belongs to and their role in each organization.

## Solution
Use the `--user` filter flag with `xbe view memberships list`:

```bash
xbe view memberships list --user <user-id>
```

## Finding the user ID
If you need to find the current authenticated user's ID:

```bash
xbe auth whoami
```

This will show the user ID along with other account details.

## Alternative: Filter with jq
If you already have JSON output, you can filter client-side:

```bash
xbe view memberships list --json | jq '.[] | select(.user_id == "<user-id>")'
```

However, using the `--user` flag is more efficient as it filters server-side.

## Understanding the output
The output shows:
- **ID**: Unique membership identifier
- **USER**: User name
- **TYPE**: Organization type (Broker, Trucker, Customer, etc.)
- **NAME**: Organization name
- **KIND**: Role type (operations/manager)

## Common patterns

### Check current user's memberships
```bash
# Get current user ID
USER_ID=$(xbe auth whoami --json | jq -r '.id')

# List their memberships
xbe view memberships list --user $USER_ID
```

### Combine with other filters
You can combine `--user` with other filters:

```bash
# User's manager roles only
xbe view memberships list --user <user-id> --kind manager

# User's memberships in a specific broker
xbe view memberships list --user <user-id> --broker <broker-id>
```
