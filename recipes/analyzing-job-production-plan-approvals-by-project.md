---
title: Analyzing job production plan approvals by project manager
when: When you need to find out who approved job production plans for a specific project manager, or when you need to analyze approval patterns across multiple plans
---

## Problem

You need to find who approved job production plans where a specific person was the project manager.

## Solution

### Step 1: Find plans managed by the project manager

First, get all job production plans where the person was the project manager during the time period:

```bash
xbe view job-production-plans list \
  --project-manager <user-id> \
  --start-at-min <start-date> \
  --start-at-max <end-date> \
  --json > /tmp/plans.json
```

### Step 2: Extract plan IDs

Save the plan IDs to a file for iteration:

```bash
jq -r '.[].id' /tmp/plans.json > /tmp/plan_ids.txt
```

### Step 3: Find approvals for each plan

For each plan, query the status changes to find approvals. **Important**: The `changed-by` field is only available in the `show` command, not in `list` output.

```bash
for plan_id in $(cat /tmp/plan_ids.txt); do
  echo "Plan: $plan_id"
  # Get approval status change
  xbe view job-production-plan-status-changes list \
    --job-production-plan $plan_id \
    --status approved \
    --json 2>&1 | \
  jq -r '.[0] | "\(.id)"' | \
  # Get full details including changed-by
  xargs -I {} xbe view job-production-plan-status-changes show {} --json 2>&1 | \
  jq -r '."changed-by"'
done
```

### Step 4: Summarize approvers

Count how many times each person approved:

```bash
for plan_id in $(cat /tmp/plan_ids.txt); do
  xbe view job-production-plan-status-changes list \
    --job-production-plan $plan_id \
    --status approved \
    --json 2>&1 | \
  jq -r '.[0] | "\(.id)"' | \
  xargs -I {} xbe view job-production-plan-status-changes show {} --json 2>&1 | \
  jq -r '."changed-by"'
done | sort | uniq -c
```

This will output something like:
```
   8 Patrick K Mulvany
   2 Jeremy Davis
   1 Travis Lewis
```

## Key Insights

- The `job-production-plan-status-changes` resource tracks all status changes including approvals
- Filter by `--status approved` to find only approval events
- The `list` command returns limited fields; use `show <id>` to get the `changed-by` field
- Project managers may approve their own plans in some workflows
