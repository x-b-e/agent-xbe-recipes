---
title: Finding who approved job production plans
when: When you need to identify which users approved specific job production plans, including finding approvers for plans managed by a specific project manager during a time period
---

## Problem

You need to find out who approved job production plans, either for specific plans or for all plans managed by a particular project manager during a time period.

## Solution

### Step 1: Find the project manager's user ID (if searching by PM name)

```bash
xbe view users list --name "<project-manager-name>" --json | jq -r '.[0].id'
```

### Step 2: Find job production plans for that PM in the time period

```bash
xbe view job-production-plans list \
  --project-manager <user-id> \
  --starts-at-after <start-date> \
  --starts-at-before <end-date> \
  --json | jq -r '.[].id'
```

### Step 3: Find approval status changes for each plan

For each plan ID, query the status changes filtered to approved status:

```bash
xbe view job-production-plan-status-changes list \
  --job-production-plan <plan-id> \
  --status approved \
  --json
```

This returns status change records with IDs but limited fields.

### Step 4: Get full details including approver

The list command doesn't return the `changed-by` field. Use `show` to get full details:

```bash
xbe view job-production-plan-status-changes show <status-change-id> --json | \
  jq '{id, status, "changed-by", "changed-by-id"}'
```

### Complete workflow for multiple plans

```bash
# Get all plan IDs
xbe view job-production-plans list \
  --project-manager <user-id> \
  --starts-at-after <start-date> \
  --starts-at-before <end-date> \
  --json | jq -r '.[].id' > /tmp/plan_ids.txt

# For each plan, find who approved it
for plan_id in $(cat /tmp/plan_ids.txt); do
  echo "Plan: $plan_id"
  xbe view job-production-plan-status-changes list \
    --job-production-plan $plan_id \
    --status approved \
    --json | \
  jq -r '.[0].id' | \
  xargs -I {} xbe view job-production-plan-status-changes show {} --json | \
  jq -r '."changed-by"'
done

# Get summary counts of approvers
for plan_id in $(cat /tmp/plan_ids.txt); do
  xbe view job-production-plan-status-changes list \
    --job-production-plan $plan_id \
    --status approved \
    --json | \
  jq -r '.[0].id' | \
  xargs -I {} xbe view job-production-plan-status-changes show {} --json | \
  jq -r '."changed-by"'
done | sort | uniq -c
```

## Key insights

- The `list` command for status changes returns limited fields; use `show` to get the `changed-by` field
- Filter status changes by `--status approved` to find approval events specifically
- Project managers may approve their own plans
- Some plans may have no approval status changes if they were never formally approved
