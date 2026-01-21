---
title: Identifying the current authenticated user
when: When you need to find out who is currently logged in to the XBE system, or when you need to get the current user's ID, email, or admin status before performing other operations
---

# Identifying the current authenticated user

To find out who is currently authenticated in the XBE system:

```bash
xbe auth whoami
```

This returns:
- User's full name
- User ID (needed for filtering memberships, etc.)
- Email address
- Mobile number
- Admin status (yes/no)

## Example output

```
Logged in as <user-name>
  ID:    <user-id>
  Email: <email>
  Mobile: <phone-number>
  Admin: <yes-or-no>
```

## Common use cases

- Getting your own user ID to use with `xbe memberships list --filter user_id=<user-id>`
- Checking if you have admin privileges before attempting admin operations
- Verifying you're logged in as the correct user
