---
title: Deleting memberships with confirmation flag
when: When you need to delete a membership and encounter the --confirm flag requirement
---

# Deleting memberships with confirmation flag

When deleting memberships, the `xbe do memberships delete` command requires a `--confirm` flag as a safety measure.

## Problem

Attempting to delete without the flag will fail:

```bash
xbe do memberships delete <membership-id>
# Error: --confirm flag is required for deletion
```

## Solution

Include the `--confirm` flag:

```bash
xbe do memberships delete <membership-id> --confirm
```

## Example

```bash
# This will fail
xbe do memberships delete 120262

# This will succeed
xbe do memberships delete 120262 --confirm
# Output: Deleted membership 120262 (Sean Devine in Quantix)
```

## Why this matters

The `--confirm` flag is a safety mechanism to prevent accidental deletions. Always include it when deleting memberships.
