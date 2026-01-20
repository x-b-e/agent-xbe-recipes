---
title: Get current user info
when: user asks who they are, what account is logged in, their user ID, or needs the current user's details
---

Use `xbe auth whoami --json` to get the logged-in user's details.

Returns: id, email, name, and organization info.

Useful when you need to:
- Filter data by the current user's ID
- Show the user their own profile info
- Verify which account is active
