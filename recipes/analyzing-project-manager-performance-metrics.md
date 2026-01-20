---
title: Analyzing project manager performance metrics
when: When you need to evaluate a project manager's performance over a date range, including goal achievement rates, surplus percentages, and comparisons to other managers
---

# Analyzing project manager performance metrics

## Overview
Project manager performance analysis requires combining data from multiple XBE resources: users, projects, job production plans, and material transactions. This recipe walks through the complete workflow.

## Step 1: Find the project manager's user ID

Search for the user by name (partial match works):

```bash
xbe view users list --name "<partial-name>" --json
```

Extract the user ID from the results. Note: The name field may not exactly match what you search for (e.g., "Patrick K Mulvany" vs "Pat Mulvaney"), so try variations if needed.

## Step 2: Get all projects managed by this person in the date range

Filter projects by project manager and job start date:

```bash
xbe view projects list --project-manager <user-id> --job-start-on-min <start-date> --job-start-on-max <end-date> --json
```

Extract the project IDs from the results.

## Step 3: Get job production plans for these projects

For each project, fetch the production plans:

```bash
xbe view job-production-plans list --project <project-id> --json
```

Filter to only "approved" plans (status field). Extract the plan IDs.

## Step 4: Analyze material transactions for each plan

For each approved plan, fetch material transactions:

```bash
xbe view material-transactions list --job-production-plan <plan-id> --json
```

## Step 5: Calculate performance metrics

For each plan, calculate:

1. **Goal Achievement**: `(total_delivered_tons / total_goal_tons) * 100`
2. **Surplus Percentage**: `((total_delivered_tons - total_used_tons) / total_delivered_tons) * 100`

Where:
- `total_delivered_tons` = sum of all transaction `net_tons`
- `total_goal_tons` = the plan's `tons` field
- `total_used_tons` = `total_delivered_tons - surplus_tons` (if surplus field exists)

Aggregate across all plans to get:
- Average goal achievement percentage
- Average surplus percentage
- Number of plans managed
- Best/worst performing days

## Step 6: Compare to other project managers

Repeat steps 2-5 for other project managers in the same organization/date range to get comparative metrics.

Rank by average goal achievement percentage.

## Tips

- Use `--json` flag throughout for easier data processing with `jq`
- The `job-production-plans` resource includes fields like `planner_id`, `project_manager_id`, `status`, and `tons`
- Material transactions include `net_tons`, `gross_tons`, `tare_tons`, and timestamps
- Filter to only "approved" plans to exclude drafts and cancelled work
- For date filtering on projects, use `--job-start-on-min` and `--job-start-on-max` rather than `--created-at` filters
