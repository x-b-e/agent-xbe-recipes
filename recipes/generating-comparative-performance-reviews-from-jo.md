---
title: Generating comparative performance reviews from job production plans
when: When you need to create a performance review for a project manager by analyzing their job production plans, comparing metrics to peers, and generating actionable feedback with specific examples
---

# Generating Comparative Performance Reviews from Job Production Plans

This recipe demonstrates how to create comprehensive performance reviews by aggregating job production plan data, calculating key metrics, benchmarking against peers, and formatting findings into actionable feedback.

## Overview

Performance reviews require:
1. Finding the user and their job production plans
2. Calculating key performance metrics
3. Identifying peer comparison groups
4. Benchmarking performance against peers
5. Finding specific examples of excellent and poor performance
6. Formatting findings into actionable recommendations

## Step 1: Identify the User

```bash
# Find the user ID by name
xbe view users list --name "<name>" --json
```

## Step 2: Get Job Production Plans for the Review Period

```bash
# List all job production plans for the user in the date range
xbe view job-production-plans list \
  --project-manager-id <user-id> \
  --start-on-min <start-date> \
  --start-on-max <end-date> \
  --json
```

## Step 3: Calculate Key Performance Metrics

For each job production plan, extract and aggregate:
- **Volume metrics**: Total tons planned vs delivered, goal achievement percentage
- **Cost metrics**: Truck spend, cost per ton
- **Time metrics**: Planned vs actual start times, delay minutes
- **Quality metrics**: Surplus percentage (planned vs actual material usage)

Example calculation with jq:

```bash
xbe view job-production-plans list \
  --project-manager-id <user-id> \
  --start-on-min <start-date> \
  --start-on-max <end-date> \
  --json | jq '
  [
    .[] | {
      date: .start_on,
      goal_tons: .goal_tons,
      delivered_tons: .delivered_tons,
      goal_pct: ((.delivered_tons / .goal_tons) * 100),
      truck_spend: .truck_spend,
      cost_per_ton: (.truck_spend / .delivered_tons),
      start_delay_minutes: .start_delay_minutes,
      surplus_pct: .surplus_percentage
    }
  ] | {
    total_plans: length,
    total_delivered: (map(.delivered_tons) | add),
    total_goal: (map(.goal_tons) | add),
    avg_goal_achievement: (map(.goal_pct) | add / length),
    avg_cost_per_ton: (map(.cost_per_ton) | add / length),
    avg_start_delay: (map(.start_delay_minutes) | add / length),
    avg_surplus: (map(.surplus_pct) | add / length)
  }'
```

## Step 4: Identify Peer Comparison Group

```bash
# Get all project managers with similar plan counts in the same period
xbe view job-production-plans list \
  --start-on-min <start-date> \
  --start-on-max <end-date> \
  --status approved \
  --json | jq '
  group_by(.project_manager_id) | 
  map({
    pm_id: .[0].project_manager_id,
    pm_name: .[0].project_manager_name,
    plan_count: length,
    total_tons: (map(.delivered_tons) | add),
    avg_goal_achievement: (map(.delivered_tons / .goal_tons * 100) | add / length),
    avg_cost_per_ton: (map(.truck_spend / .delivered_tons) | add / length),
    avg_start_delay: (map(.start_delay_minutes) | add / length),
    avg_surplus: (map(.surplus_percentage) | add / length)
  }) | 
  sort_by(.plan_count)'
```

## Step 5: Rank Performance Against Peers

For each metric, rank the subject against their peer group:

```bash
# Example: Ranking by cost efficiency
xbe view job-production-plans list \
  --start-on-min <start-date> \
  --start-on-max <end-date> \
  --status approved \
  --json | jq '
  group_by(.project_manager_id) | 
  map({
    pm_name: .[0].project_manager_name,
    plan_count: length,
    avg_cost_per_ton: (map(.truck_spend / .delivered_tons) | add / length)
  }) | 
  sort_by(.avg_cost_per_ton) | 
  to_entries | 
  map(.value + {rank: .key + 1})'
```

## Step 6: Find Specific Examples

### Excellent Performance Examples

Find days where the subject exceeded goals significantly:

```bash
xbe view job-production-plans list \
  --project-manager-id <user-id> \
  --start-on-min <start-date> \
  --start-on-max <end-date> \
  --json | jq '
  map(select(.delivered_tons / .goal_tons > 1.05)) | 
  sort_by(.delivered_tons / .goal_tons) | 
  reverse | 
  .[:5]'
```

### Areas Needing Improvement

Find days with significant shortfalls:

```bash
xbe view job-production-plans list \
  --project-manager-id <user-id> \
  --start-on-min <start-date> \
  --start-on-max <end-date> \
  --json | jq '
  map(select(.delivered_tons / .goal_tons < 0.90)) | 
  sort_by(.delivered_tons / .goal_tons) | 
  .[:5]'
```

## Step 7: Find Better-Performing Peers for Each Improvement Area

For each area of improvement, identify peers who perform better:

```bash
# Find peers with better start time performance
xbe view job-production-plans list \
  --start-on-min <start-date> \
  --start-on-max <end-date> \
  --json | jq --arg target_delay "<target-delay>" '
  group_by(.project_manager_id) | 
  map({
    pm_name: .[0].project_manager_name,
    plan_count: length,
    avg_start_delay: (map(.start_delay_minutes) | add / length)
  }) | 
  map(select(.plan_count >= 20 and .plan_count <= 35)) | 
  map(select(.avg_start_delay < ($target_delay | tonumber))) | 
  sort_by(.avg_start_delay)'
```

## Step 8: Format the Review Script

Create a structured document with:

1. **Overall Performance Summary**: Key metrics and rankings
2. **Three Areas of Excellence**: 
   - Metric achievement and ranking
   - Why it's excellent (peer comparisons)
   - Specific examples with dates and numbers
   - Business impact quantification
3. **Three Areas for Improvement**:
   - Current performance and ranking
   - The opportunity (gap to better performers)
   - Comparison to better-performing peers
   - What better performers do differently
   - Specific examples from subject's data
   - Actionable recommendations with targets
4. **Development Plan**: 30-day, 90-day, and 12-month goals

## Tips

- **Peer grouping**: Compare against PMs with similar plan counts (±5 plans) for fair comparison
- **Multiple metrics**: Consider goal achievement, cost efficiency, start time punctuality, and surplus management
- **Specific examples**: Always include dates, actual numbers, and percentages
- **Actionable targets**: Set specific, measurable improvement goals
- **Balance**: Show both strengths and opportunities - don't focus only on problems
- **Business impact**: Quantify the impact in dollars, time saved, or customer satisfaction
- **Pattern analysis**: Look for patterns in underperformance (e.g., milling jobs, specific sites, time of month)

## Common Pitfalls

- **Unfair comparisons**: Don't compare PMs with vastly different workloads
- **Cherry-picking**: Show representative examples, not just outliers
- **Vague feedback**: Always include specific numbers and dates
- **No actionability**: Every improvement area needs concrete next steps
- **Ignoring context**: Consider external factors (weather, equipment, customer changes)
