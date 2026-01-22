---
title: Generating employee performance reviews from job production plan data
when: When you need to create a performance review for a project manager or employee based on their job production plan performance, including identifying strengths and areas for improvement by comparing to peers
---

# Generating employee performance reviews from job production plan data

## Overview
This recipe shows how to generate comprehensive performance reviews for project managers by analyzing their job production plan data and comparing against peers.

## Key Metrics to Analyze
- **Goal Achievement %**: Actual tons delivered vs planned tons
- **Cost Efficiency**: Truck spend per ton (lower is better)
- **Surplus Control**: Actual surplus % (lower is better, indicates less material waste)
- **On-Time Starts**: Average delay from planned start time

## Step-by-Step Workflow

### 1. Find the employee's user ID
```bash
xbe view users list --name "<employee-name>"
```

### 2. Get all job production plans for the employee in the review period
```bash
xbe view job-production-plans list \
  --project-manager-id <user-id> \
  --start-on-from <start-date> \
  --start-on-to <end-date> \
  --json > employee_plans.json
```

### 3. Calculate key performance metrics

#### Goal Achievement Percentage
```bash
jq '
  [.[] | select(.tons_delivered != null and .planned_tons != null)] |
  {
    total_delivered: map(.tons_delivered) | add,
    total_planned: map(.planned_tons) | add
  } |
  .goal_percentage = ((.total_delivered / .total_planned) * 100)
' employee_plans.json
```

#### Average Start Time Delay
```bash
jq '
  [.[] | select(.actual_start_at != null and .planned_start_at != null) |
    (.actual_start_at | fromdateiso8601) - (.planned_start_at | fromdateiso8601)
  ] |
  (add / length / 60) as $avg_delay_minutes |
  {average_delay_minutes: $avg_delay_minutes}
' employee_plans.json
```

#### Cost Per Ton
```bash
jq '
  [.[] | select(.truck_spend != null and .tons_delivered != null and .tons_delivered > 0)] |
  {
    total_truck_spend: map(.truck_spend) | add,
    total_tons: map(.tons_delivered) | add
  } |
  .cost_per_ton = (.total_truck_spend / .total_tons)
' employee_plans.json
```

#### Average Surplus Percentage
```bash
jq '
  [.[] | select(.actual_surplus_percentage != null)] |
  (map(.actual_surplus_percentage) | add / length) as $avg_surplus |
  {average_surplus_percentage: $avg_surplus}
' employee_plans.json
```

### 4. Get peer comparison data

To compare against peers, you need to:

a) Identify the peer group (e.g., all PMs from the same organization with minimum plan threshold)

b) Query plans for each peer using the same date range:
```bash
# For each peer PM
xbe view job-production-plans list \
  --project-manager-id <peer-user-id> \
  --start-on-from <start-date> \
  --start-on-to <end-date> \
  --json > peer_<peer-name>_plans.json
```

c) Calculate the same metrics for each peer

### 5. Find specific examples of excellent performance

#### Best single days (highest goal achievement)
```bash
jq '
  [.[] | select(.tons_delivered != null and .planned_tons != null) |
    . + {goal_pct: ((.tons_delivered / .planned_tons) * 100)}
  ] |
  sort_by(.goal_pct) | reverse | .[0:5] |
  map({
    date: .start_on,
    tons_delivered: .tons_delivered,
    planned_tons: .planned_tons,
    goal_pct: .goal_pct,
    job_site: .job_site_name
  })
' employee_plans.json
```

#### Days with best cost efficiency
```bash
jq '
  [.[] | select(.truck_spend != null and .tons_delivered != null and .tons_delivered > 0) |
    . + {cost_per_ton: (.truck_spend / .tons_delivered)}
  ] |
  sort_by(.cost_per_ton) | .[0:5] |
  map({
    date: .start_on,
    cost_per_ton: .cost_per_ton,
    tons_delivered: .tons_delivered,
    job_site: .job_site_name
  })
' employee_plans.json
```

#### Days with best surplus control
```bash
jq '
  [.[] | select(.actual_surplus_percentage != null)] |
  sort_by(.actual_surplus_percentage) | .[0:5] |
  map({
    date: .start_on,
    actual_surplus_pct: .actual_surplus_percentage,
    approved_surplus_pct: .approved_surplus_percentage,
    tons_delivered: .tons_delivered,
    job_site: .job_site_name
  })
' employee_plans.json
```

### 6. Identify areas for improvement

#### Worst performing days (lowest goal achievement)
```bash
jq '
  [.[] | select(.tons_delivered != null and .planned_tons != null) |
    . + {goal_pct: ((.tons_delivered / .planned_tons) * 100)}
  ] |
  sort_by(.goal_pct) | .[0:5] |
  map({
    date: .start_on,
    tons_delivered: .tons_delivered,
    planned_tons: .planned_tons,
    goal_pct: .goal_pct,
    job_site: .job_site_name
  })
' employee_plans.json
```

#### Days with most delay
```bash
jq '
  [.[] | select(.actual_start_at != null and .planned_start_at != null) |
    . + {
      delay_minutes: ((.actual_start_at | fromdateiso8601) - (.planned_start_at | fromdateiso8601)) / 60
    }
  ] |
  sort_by(.delay_minutes) | reverse | .[0:5] |
  map({
    date: .start_on,
    delay_minutes: .delay_minutes,
    job_site: .job_site_name
  })
' employee_plans.json
```

### 7. Create comparison table

Combine all peer data into a comparison table:
```bash
jq -s '
  map({
    name: .[0].project_manager_name,
    plans: length,
    goal_pct: ([.[] | select(.tons_delivered != null and .planned_tons != null)] | 
      (map(.tons_delivered) | add) / (map(.planned_tons) | add) * 100),
    avg_delay: ([.[] | select(.actual_start_at != null and .planned_start_at != null) |
      ((.actual_start_at | fromdateiso8601) - (.planned_start_at | fromdateiso8601))]
      | add / length / 60),
    cost_per_ton: ([.[] | select(.truck_spend != null and .tons_delivered != null and .tons_delivered > 0)] |
      (map(.truck_spend) | add) / (map(.tons_delivered) | add)),
    avg_surplus: ([.[] | select(.actual_surplus_percentage != null)] |
      map(.actual_surplus_percentage) | add / length)
  }) |
  sort_by(.goal_pct) | reverse
' employee_plans.json peer_*.json
```

## Tips

- **Minimum plan threshold**: Filter peer comparison to PMs with at least 15-20 plans to ensure statistical significance
- **Same organization**: Compare within the same broker/organization for fair comparison
- **Date alignment**: Use exact same date ranges for all comparisons
- **Outlier analysis**: For days that significantly underperformed, investigate root causes (weather, equipment, crew issues)
- **Pattern recognition**: Look for patterns in underperformance (e.g., specific job types, time of month, material types)
- **Business impact calculation**: Calculate dollar savings/costs by comparing employee's rates to peer averages multiplied by total volume

## Performance Review Structure

1. **Overall Summary**: Total plans, volume, and rankings vs peers
2. **Three Areas of Excellence**: Top 3 metrics with specific examples and business impact
3. **Three Areas for Improvement**: Bottom 3 metrics with peer comparisons and actionable recommendations
4. **Development Plan**: 30/90/180/365-day goals based on the analysis
5. **Business Impact**: Quantify cost savings or losses compared to peer performance
