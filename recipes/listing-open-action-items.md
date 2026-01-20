---
title: Listing open action items
when: When you need to view action items that are not yet completed, including items in various states like in_progress, ready_for_work, editing, etc.
---

# Listing open action items

## Discovery

The `xbe view action-items list` command provides access to trackable work items like tasks, bugs, features, and integrations.

```bash
xbe view action-items list --help
```

## Listing all open (non-completed) items

To see all action items that are still active, filter by all non-complete statuses:

```bash
xbe view action-items list --status editing,ready_for_work,in_progress,in_verification,on_hold
```

This excludes only items with status `complete`.

## Available statuses

- `editing` - Being drafted
- `ready_for_work` - Ready to start
- `in_progress` - Currently being worked on
- `in_verification` - Being verified/tested
- `complete` - Finished
- `on_hold` - Paused

## Common filters

```bash
# Only in-progress items
xbe view action-items list --status in_progress

# In-progress or ready to work
xbe view action-items list --status in_progress,ready_for_work

# By kind (type of work)
xbe view action-items list --kind feature,bug_fix

# Combine filters (AND logic across filter types)
xbe view action-items list --status in_progress --kind feature

# By project
xbe view action-items list --project <project-id>

# By responsible person
xbe view action-items list --responsible-person <user-id>
```

## Output formats

```bash
# Table format (default)
xbe view action-items list --status in_progress

# JSON format for parsing
xbe view action-items list --status in_progress --json
```

## Getting details

Once you have an action item ID, get full details:

```bash
xbe view action-items show <action-item-id>
```
