---
title: Creating project manager performance reviews from job production plans
when: When you need to create a performance review for a project manager based on their job production plan data, comparing them to peers and analyzing multiple performance dimensions
---

## Overview

This recipe shows how to build a comprehensive performance review for a project manager using job production plan data from the XBE platform. The workflow aggregates metrics across multiple dimensions and compares performance to organizational peers.

## Step 1: Identify the User and Organization

First, find the user being reviewed:

```bash
xbe view users list --name "<manager-name>"
```

Note the user ID and organization (broker) from the results.

## Step 2: Query Job Production Plans for the Review Period

Retrieve all job production plans for the date range:

```bash
xbe view job-production-plans list \
  --broker <broker-id> \
  --start-on <start-date> \
  --end-on <end-date> \
  --json > plans.json
```

Filter for the specific project manager:

```bash
jq '[.[] | select(.project_manager_id == <pm-id>)]' plans.json > pm_plans.json
```

## Step 3: Extract and Aggregate Key Metrics

### Count Total Plans

```bash
jq 'length' pm_plans.json
```

### Calculate Goal Achievement

Sum actual vs planned tonnage:

```bash
jq '[.[] | {actual: .actual_tons, goal: .goal_tons}] | 
    {total_actual: (map(.actual) | add), 
     total_goal: (map(.goal) | add)} | 
    .achievement_pct = ((.total_actual / .total_goal) * 100)' pm_plans.json
```

### Calculate Average Start Time Delay

```bash
jq '[.[] | .actual_start_delay_minutes] | add / length' pm_plans.json
```

### Calculate Cost Efficiency (Cost per Ton)

```bash
jq '{total_cost: (map(.truck_spend) | add), 
     total_tons: (map(.actual_tons) | add)} | 
    .cost_per_ton = (.total_cost / .total_tons)' pm_plans.json
```

### Calculate Surplus Percentage

```bash
jq '{actual_surplus: (map(.actual_surplus_tons) | add), 
     total_tons: (map(.actual_tons) | add)} | 
    .surplus_pct = ((.actual_surplus / .total_tons) * 100)' pm_plans.json
```

## Step 4: Compare to Peer Project Managers

Get all project managers in the organization with significant activity:

```bash
jq 'group_by(.project_manager_id) | 
    map({pm_id: .[0].project_manager_id, 
         pm_name: .[0].project_manager_name, 
         plan_count: length, 
         goal_pct: ((map(.actual_tons) | add) / (map(.goal_tons) | add) * 100), 
         avg_delay: ((map(.actual_start_delay_minutes) | add) / length), 
         cost_per_ton: ((map(.truck_spend) | add) / (map(.actual_tons) | add)), 
         surplus_pct: ((map(.actual_surplus_tons) | add) / (map(.actual_tons) | add) * 100)}) | 
    map(select(.plan_count >= 15))' plans.json
```

## Step 5: Identify Specific Examples

### Best Performance Days (Goal Achievement)

```bash
jq '[.[] | {date: .start_on, 
            goal_pct: ((.actual_tons / .goal_tons) * 100), 
            actual: .actual_tons, 
            goal: .goal_tons}] | 
    sort_by(.goal_pct) | reverse | .[0:5]' pm_plans.json
```

### Days with Shortfalls

```bash
jq '[.[] | {date: .start_on, 
            goal_pct: ((.actual_tons / .goal_tons) * 100), 
            actual: .actual_tons, 
            goal: .goal_tons}] | 
    map(select(.goal_pct < 90)) | 
    sort_by(.goal_pct)' pm_plans.json
```

### Best Cost Efficiency Days

```bash
jq '[.[] | {date: .start_on, 
            cost_per_ton: (.truck_spend / .actual_tons), 
            tons: .actual_tons, 
            spend: .truck_spend}] | 
    sort_by(.cost_per_ton) | .[0:5]' pm_plans.json
```

### Best Surplus Control Days

```bash
jq '[.[] | {date: .start_on, 
            surplus_pct: ((.actual_surplus_tons / .actual_tons) * 100), 
            surplus: .actual_surplus_tons, 
            approved: .approved_surplus_tons}] | 
    sort_by(.surplus_pct) | .[0:5]' pm_plans.json
```

## Step 6: Structure the Review

A comprehensive performance review should include:

1. **Overall Performance Summary**: Total plans, volume, and key metric averages
2. **Peer Rankings**: Where the PM ranks among peers in each metric
3. **Three Areas of Excellence**: Top metrics with specific examples and business impact
4. **Three Areas for Improvement**: Metrics with room for growth, with peer comparisons
5. **Development Plan**: Specific, time-bound goals and action items

## Key Metrics to Track

- **Goal Achievement %**: Actual tons / Goal tons
- **Start Time Delay**: Average minutes past planned start
- **Cost Efficiency**: Truck spend / Actual tons ($/ton)
- **Surplus Control**: Actual surplus / Actual tons (%)
- **Volume**: Total plans managed and total tons delivered

## Tips

- Filter peer comparison to PMs with 15+ plans to ensure statistical significance
- Look for patterns in underperformance (specific job types, time periods, sites)
- Calculate business impact in dollar terms for cost and surplus metrics
- Use concrete examples from specific dates to illustrate both strengths and improvement areas
- Frame improvement areas constructively, especially for top performers
- Set specific, measurable targets for the next review period
