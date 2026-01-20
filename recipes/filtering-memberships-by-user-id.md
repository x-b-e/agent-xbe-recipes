---
title: Filtering memberships by user
when: When you need to see memberships for a specific user, or when a user asks for 'my memberships' and you need to identify who they are first
---

# Filtering memberships by user

## Overview
When a user asks for "my memberships" or you need to see memberships for a specific user, you need to filter by user ID.

## Getting the current user's memberships

### Step 1: Identify the current user
```bash
xbe auth whoami
```

This returns:
```
Logged in as <user-name>
  ID:    <user-id>
  Email: <email>
  Mobile: <phone>
  Admin: yes/no
```

### Step 2: List memberships for that user
```bash
xbe view memberships list --user <user-id>
```

## Output format
The filtered results show only memberships for the specified user:
```
                     ORGANIZATION                  
ID      USER         TYPE          NAME            KIND
<id>    <user-name>  Broker        <org-name>      operations
<id>    <user-name>  Broker        <org-name>      manager
```

## Related
- See "Listing user memberships" for the basic list command without filtering
- See "Deleting memberships" for removing memberships
- See "Creating memberships" for adding new memberships
