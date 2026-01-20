---
title: Deleting memberships
when: When you need to delete a membership (remove a user's association with an organization)
---

# Deleting memberships

## Overview
Delete a membership to remove a user's association with an organization.

## Important
The delete command requires the `--confirm` flag for safety. If you forget it, you'll get an error:
```
Exit code 1
--confirm flag is required for deletion
```

## Command
```bash
xbe do memberships delete <membership-id> --confirm
```

## Example workflow
1. First, identify the membership ID to delete (e.g., by listing user memberships)
2. Delete with confirmation:
```bash
xbe do memberships delete <membership-id> --confirm
```

## Success output
```
Deleted membership <membership-id> (<user-name> in <organization-name>)
```
