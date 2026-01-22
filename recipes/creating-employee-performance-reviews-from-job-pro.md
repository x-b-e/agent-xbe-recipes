---
title: Creating employee performance reviews from job production plan data
when: When you need to create a performance review for an employee (project manager, foreman, etc.) based on their job production plan performance, including identifying strengths with examples and areas for improvement with peer comparisons
---

# Creating Employee Performance Reviews from Job Production Plan Data

This recipe shows how to create comprehensive performance reviews for employees based on their job production plan performance.

## Overview

The process involves:
1. Identifying the employee and time period
2. Gathering their job production plan data
3. Calculating key performance metrics
4. Comparing performance to peers
5. Identifying specific examples of excellence and areas for improvement
6. Creating a formatted review document

## Step 1: Identify the Employee

First, find the employee's user ID:

```bash
xbe view users list --name "<employee-name>" --json
```

## Step 2: Gather Job Production Plans

Retrieve all approved plans for the employee in the target time period:

```bash
xbe view job-production-plans list \
  --project-manager-id <user-id> \
  --status approved \
  --start-on-from <start-date> \
  --start-on-to <end-date> \
  --json
```

## Step 3: Calculate Performance Metrics

For each plan, extract key metrics:
- Total tonnage delivered vs goal
- Cost per ton (truck spend / tonnage)
- Start time vs planned start time
- Surplus percentage (actual vs approved)

Aggregate across all plans:

```bash
# Example: Calculate total tonnage and goal achievement
jq '[.[] | {delivered: .tons_delivered, goal: .tons_goal}] | 
    {total_delivered: (map(.delivered) | add), 
     total_goal: (map(.goal) | add)} | 
    .achievement_pct = (.total_delivered / .total_goal * 100)' plans.json
```

## Step 4: Get Peer Comparison Data

Retrieve plans from all project managers in the same period to establish rankings:

```bash
# Get all PMs' plans for comparison
xbe view job-production-plans list \
  --status approved \
  --start-on-from <start-date> \
  --start-on-to <end-date> \
  --json > all_plans.json
```

Group by project manager and calculate the same metrics:

```bash
jq 'group_by(.project_manager.id) | 
    map({
      pm_id: .[0].project_manager.id,
      pm_name: .[0].project_manager.name,
      plan_count: length,
      total_delivered: (map(.tons_delivered) | add),
      total_goal: (map(.tons_goal) | add),
      avg_delay: (map(.start_time_delay_minutes) | add / length),
      avg_surplus: (map(.surplus_pct) | add / length),
      total_truck_spend: (map(.truck_spend) | add)
    }) | 
    map(. + {goal_achievement: (.total_delivered / .total_goal * 100)}) |
    map(. + {cost_per_ton: (.total_truck_spend / .total_delivered)})' \
    all_plans.json > pm_metrics.json
```

## Step 5: Identify Similar Peers

Filter for PMs with similar plan volumes for fair comparison:

```bash
# Find PMs with similar plan counts (e.g., within +/- 10 plans)
jq --arg target_count "<plan-count>" \
   'map(select(.plan_count >= ($target_count | tonumber) - 10 and 
               .plan_count <= ($target_count | tonumber) + 10))' \
   pm_metrics.json > similar_peers.json
```

## Step 6: Rank Performance

For each metric, rank the target employee against all PMs:

```bash
# Rank by goal achievement (descending)
jq 'sort_by(-.goal_achievement) | 
    to_entries | 
    map({rank: .key + 1, pm: .value})' pm_metrics.json

# Rank by cost per ton (ascending - lower is better)
jq 'sort_by(.cost_per_ton) | 
    to_entries | 
    map({rank: .key + 1, pm: .value})' pm_metrics.json
```

## Step 7: Identify Specific Examples

For strengths, find best-performing days:

```bash
# Find days that exceeded goal by most
jq '[.[] | select(.tons_delivered > .tons_goal) | 
    {date: .start_on, 
     site: .material_site.name, 
     delivered: .tons_delivered, 
     goal: .tons_goal, 
     achievement: (.tons_delivered / .tons_goal * 100)}] | 
    sort_by(-.achievement) | .[0:5]' plans.json
```

For improvement areas, find worst-performing days:

```bash
# Find days with significant shortfalls
jq '[.[] | select(.tons_delivered < .tons_goal) | 
    {date: .start_on, 
     delivered: .tons_delivered, 
     goal: .tons_goal, 
     achievement: (.tons_delivered / .tons_goal * 100)}] | 
    sort_by(.achievement) | .[0:5]' plans.json
```

## Step 8: Find Better Performers for Each Improvement Area

For each identified weakness, find peers who excel in that dimension:

```bash
# Find PMs with better start times among similar-volume peers
jq --arg target_id "<user-id>" \
   'map(select(.pm_id != $target_id)) | 
    sort_by(.avg_delay) | .[0:5] | 
    map({name: .pm_name, plans: .plan_count, avg_delay: .avg_delay})' \
   similar_peers.json
```

## Step 9: Create Review Document

Structure the review with:

1. **Overall Performance Summary**
   - Key metrics for the period
   - Rankings among all PMs
   - Plan count and total volume

2. **Three Areas of Excellence** (for each):
   - Achievement with specific ranking
   - Why this is excellent (peer comparisons showing how they beat others)
   - Specific examples from their actual work with dates and numbers
   - Business impact quantification

3. **Three Areas for Improvement** (for each):
   - Current performance with ranking
   - The opportunity (what improvement would mean)
   - Comparison to better performers with specific names and metrics
   - What better performers do differently (analysis)
   - Specific examples of shortfalls from their work
   - Recommendations with measurable targets

4. **Development Plan**
   - Immediate actions (30 days)
   - 90-day goals
   - 12-month vision

Write to a markdown file:

```bash
cat > /tmp/review_<employee-name>.md << 'EOF'
# PERFORMANCE REVIEW SCRIPT
## <Employee Name> - <Period>

### MEETING OVERVIEW
This review covers <Employee Name>'s performance as <Role> during <Period>.

---

## OVERALL PERFORMANCE SUMMARY

### Key Metrics (<Period>)
- **Plans Completed:** <count> approved job production plans
- **Total Volume:** <tons> tons delivered (Goal: <goal-tons> tons)
- **Goal Achievement:** <percentage>% of planned tonnage
- **Cost Efficiency:** $<amount> per ton (truck spend)
- **Average Start Time:** <minutes> minutes delay from planned start
- **Surplus Management:** <percentage>% actual surplus

### Rankings Among <Role>s (<filter-description>)
- **Goal Achievement:** #<rank> of <total> PMs
- **Cost Efficiency:** #<rank> of <total> PMs  
- **On-Time Starts:** #<rank> of <total> PMs
- **Surplus Control:** #<rank> of <total> PMs

---

## THREE AREAS OF EXCELLENT PERFORMANCE

### 1. <STRENGTH NAME>
**Achievement:** <metric-value> - ranked #<rank> out of <total> PMs

**Why This is Excellent:**
- <Comparison to peers>
- <Percentage or absolute difference from others>

**Specific Example:**
- <Concrete example from their work with date, location, numbers>

**Business Impact:**
<Quantification of what this excellence means for the business>

---

## THREE AREAS FOR IMPROVEMENT

### 1. <IMPROVEMENT AREA NAME>
**Current Performance:** <current-metric> - ranked #<rank> of <total> PMs

**The Opportunity:**
<What improvement would mean>

**Comparison to Better Performers:**
- **<Peer Name>** (<similar-context>): <their-better-metric>
- **<Peer Name>** (<similar-context>): <their-better-metric>

**What Better Performers Do Differently:**
<Analysis of what makes them better>

**Specific Examples from <Employee>'s <Period>:**
- <Date>: <specific shortfall with numbers>

**Recommendation:**
- <Specific action>
- Target: <measurable goal>

---

## DEVELOPMENT PLAN

### Immediate Actions (Next 30 Days)
1. <Action 1>
2. <Action 2>
3. <Action 3>

### 90-Day Goals
1. <Goal 1>
2. <Goal 2>
3. <Goal 3>

### 12-Month Vision
- <Vision element 1>
- <Vision element 2>

---

**Prepared by:** <Reviewer Name>
**Date:** <Review Date>
**Review Period:** <Period>
EOF
```

## Key Patterns

- **Always use similar-volume peers** for fair comparisons (e.g., PMs with 20-35 plans)
- **Quantify business impact** for strengths (e.g., "saved $X compared to peers")
- **Provide specific dates and numbers** for all examples
- **Show what better performers do differently** with analysis, not just metrics
- **Set measurable targets** for improvement areas
- **Balance positive and constructive** - three strengths, three improvements

## Common Metrics for Project Managers

- **Goal Achievement %**: tons_delivered / tons_goal * 100
- **Cost per Ton**: truck_spend / tons_delivered
- **Start Time Delay**: actual_start_time - planned_start_time (in minutes)
- **Surplus %**: (actual_surplus / approved_surplus) * 100 or actual_surplus percentage
- **Plan Count**: Number of approved plans in period

## Tips

- Filter for `--status approved` to only include completed plans
- Use `--json` output for programmatic analysis with jq
- Consider seasonal factors (e.g., weather in October vs July)
- Look for patterns (e.g., "milling jobs consistently under goal")
- Provide context for outliers (e.g., equipment failure, weather delay)
- Include a mix of absolute rankings and peer-group rankings
