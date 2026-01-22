---
title: Setting custom membership titles
when: When you need to set a custom title for a user's membership within an organization beyond the standard operations/manager roles
---

# Setting custom membership titles

## Overview
Memberships support custom titles that can be set independently of the role (operations/manager). This allows for personalized titles like "Chief Yahoo", "Lead Engineer", etc.

## Command Pattern
```bash
xbe do memberships update <membership-id> --title "<custom-title>"
```

## Example
```bash
# Set a custom title for membership
xbe do memberships update <membership-id> --title "Chief Yahoo"
```

## Notes
- The title is independent of the role kind (operations/manager)
- Titles can be any string value
- To view the updated title, use `xbe view memberships show <membership-id>`
- The title field is separate from admin status and role permissions
