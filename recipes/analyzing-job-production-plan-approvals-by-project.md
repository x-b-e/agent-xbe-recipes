---
title: Analyzing job production plan approvals by project manager
when: When you need to find out who approved job production plans for a specific project manager, or when you need to analyze approval patterns across multiple plans
---

## Pattern

To find who approved job production plans for a specific project manager:

1. Find the project manager's user ID
2. Get their job production plans for the time period
3. For each plan, find the approval status change record
4. Extract the approver from the status change record

## Key Insight

The approver information is stored in the `job-production-plan-status-changes` resource under the `changed-by` field. This field is **only available when using `show`**, not when using `list`. You must:

1. Use `list` to find the status change ID
2. Use `show` with that ID to get the `changed-by` field

## Example

```bash
# 1. Find the project manager's user ID
xbe view users list --name-cont "<manager-name>" --json | jq -r '.[0].id'

# 2. Get their job production plans for a date range
xbe view job-production-plans list \
  --start-on-min <start-date> \
  --start-on-max <end-date> \
  --project-manager <user-id> \
  --json \
  --limit 500 > /tmp/plans.json

# 3. Extract plan IDs
jq -r '.[].id' /tmp/plans.json > /tmp/plan_ids.txt

# 4. For each plan, find who approved it
# Note: Must use 'show' to get 'changed-by' field
for plan_id in $(cat /tmp/plan_ids.txt); do
  echo "Plan: $plan_id"
  xbe view job-production-plan-status-changes list \
    --job-production-plan $plan_id \
    --status approved \
    --json 2>&1 | jq -r '.[0] | "\(.id)"' | \
    xargs -I {} xbe view job-production-plan-status-changes show {} --json 2>&1 | \
    jq -r '."changed-by"'
done

# 5. Get a summary count of approvers
for plan_id in $(cat /tmp/plan_ids.txt); do
  xbe view job-production-plan-status-changes list \
    --job-production-plan $plan_id \
    --status approved \
    --json 2>&1 | jq -r '.[0] | "\(.id)"' | \
    xargs -I {} xbe view job-production-plan-status-changes show {} --json 2>&1 | \
    jq -r '."changed-by"'
done | sort | uniq -c
```

## Common Patterns

### Approving own plans
It's common for project managers to approve their own plans. In the data, you may see the same person as both `project-manager` and `changed-by`.

### Multiple time periods
To compare approval patterns across different months:

```bash
# Query plans for each month separately
for month in 10 11 12; do
  echo "Month: $month"
  xbe view job-production-plans list \
    --start-on-min 2025-${month}-01 \
    --start-on-max 2025-${month}-31 \
    --project-manager <user-id> \
    --json --limit 500 | \
    jq -r '.[].id' | \
    while read plan_id; do
      xbe view job-production-plan-status-changes list \
        --job-production-plan $plan_id \
        --status approved \
        --json 2>&1 | jq -r '.[0] | "\(.id)"' | \
        xargs -I {} xbe view job-production-plan-status-changes show {} --json 2>&1 | \
        jq -r '."changed-by"'
    done | sort | uniq -c
done
```
