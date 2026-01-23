---
title: Analyzing project manager performance with peer comparisons
when: When you need to evaluate a PM's performance for a review meeting, including narrative strengths/weaknesses, mathematical comparisons to peers, specific examples, and mentorship recommendations
---

## Overview

This recipe shows how to conduct comprehensive PM performance analysis by comparing their job production plans against peer averages across multiple dimensions.

## Steps

### 1. Identify the PM and date range

First, find the PM's user ID and define the evaluation period:

```bash
xbe view users list --json | jq '.[] | select(.full_name | contains("<pm-name>"))'
```

### 2. Fetch all relevant job production plans

Get all Superior Bowen (or relevant broker) plans for the period:

```bash
xbe summarize job-production-plan-summary create \
  --broker-id <broker-id> \
  --start-on <start-date> \
  --end-on <end-date> \
  --group-by job_production_plan_id \
  --json > /tmp/all_plans.json
```

### 3. Enrich with PM information

Fetch individual plan details to get PM names (summaries don't include this):

```bash
# Extract all unique plan IDs
jq -r '.[].job_production_plan_id' /tmp/all_plans.json > /tmp/plan_ids.txt

# Fetch each plan's details
while read plan_id; do
  xbe view job-production-plans show "$plan_id" --json
  sleep 0.1  # Rate limiting
done < /tmp/plan_ids.txt | jq -s '.' > /tmp/plan_details.json

# Merge summary data with PM names
jq -s '.[0] as $summaries | .[1] as $details |
  $summaries | map(
    . as $summary |
    ($details[] | select(.id == $summary.job_production_plan_id)) as $detail |
    $summary + {
      project_manager: $detail.project_manager.full_name,
      planner: $detail.planner.full_name,
      job_name: $detail.job.name
    }
  )' /tmp/all_plans.json /tmp/plan_details.json > /tmp/enriched_plans.json
```

### 4. Calculate key metrics

Use Python to compute comparative statistics:

```python
import json
import statistics

with open('/tmp/enriched_plans.json', 'r') as f:
    all_plans = json.load(f)

# Filter for target PM and peers
target_pm = "<pm-full-name>"
target_plans = [p for p in all_plans if p['project_manager'] == target_pm and p['status'] == 'approved']
all_approved = [p for p in all_plans if p['status'] == 'approved']

# Calculate surplus control variance
def calc_surplus_variance(plans):
    variances = []
    for p in plans:
        if p.get('approved_surplus_pct') is not None and p.get('actual_surplus_pct') is not None:
            variance = abs((p['actual_surplus_pct'] - p['approved_surplus_pct']) * 100)
            variances.append(variance)
    return statistics.mean(variances) if variances else None

target_surplus_var = calc_surplus_variance(target_plans)
avg_surplus_var = calc_surplus_variance(all_approved)

# Calculate goal achievement
def calc_goal_achievement(plans):
    achievements = [p['pct_goal'] for p in plans if p.get('pct_goal') is not None]
    return statistics.mean(achievements) if achievements else None

target_goal = calc_goal_achievement(target_plans)
avg_goal = calc_goal_achievement(all_approved)

# Calculate cancellation rate
target_cancelled = len([p for p in all_plans if p['project_manager'] == target_pm and p['status'] == 'cancelled'])
target_total = len([p for p in all_plans if p['project_manager'] == target_pm])
target_cancel_rate = (target_cancelled / target_total * 100) if target_total > 0 else 0

all_cancelled = len([p for p in all_plans if p['status'] == 'cancelled'])
all_total = len(all_plans)
avg_cancel_rate = (all_cancelled / all_total * 100) if all_total > 0 else 0

print(f"Surplus Control Variance: {target_surplus_var:.1f}% vs {avg_surplus_var:.1f}% avg")
print(f"Goal Achievement: {target_goal:.1f}% vs {avg_goal:.1f}% avg")
print(f"Cancellation Rate: {target_cancel_rate:.1f}% vs {avg_cancel_rate:.1f}% avg")
```

### 5. Identify specific examples

Extract concrete plan IDs for discussion:

```python
# Best surplus control examples
approved_with_surplus = [p for p in target_plans 
                         if p.get('approved_surplus_pct') is not None 
                         and p.get('actual_surplus_pct') is not None]

surplus_examples = []
for p in approved_with_surplus:
    variance = abs((p['actual_surplus_pct'] - p['approved_surplus_pct']) * 100)
    surplus_examples.append({
        'id': p['id'],
        'date': p['start_on'],
        'variance': variance,
        'approved': p['approved_surplus_pct'] * 100,
        'actual': p['actual_surplus_pct'] * 100
    })

surplus_examples.sort(key=lambda x: x['variance'])

print("\nBest Surplus Control:")
for ex in surplus_examples[:3]:
    print(f"  Plan {ex['id']} ({ex['date']}): {ex['variance']:.1f}% variance")

print("\nWorst Surplus Control:")
for ex in reversed(surplus_examples[-3:]):
    print(f"  Plan {ex['id']} ({ex['date']}): {ex['variance']:.1f}% variance")

# Best goal achievement
with_goals = [p for p in target_plans if p.get('pct_goal') is not None]
with_goals.sort(key=lambda x: x['pct_goal'], reverse=True)

print("\nBest Goal Achievement:")
for p in with_goals[:5]:
    print(f"  Plan {p['id']} ({p['start_on']}): {p['pct_goal']:.1f}%")

print("\nWorst Goal Achievement:")
for p in with_goals[-3:]:
    print(f"  Plan {p['id']} ({p['start_on']}): {p['pct_goal']:.1f}%")
```

### 6. Generate peer comparison for mentorship

Identify who excels in areas where the target PM struggles:

```python
# Group by PM
from collections import defaultdict

pm_stats = defaultdict(lambda: {
    'plans': [],
    'surplus_vars': [],
    'goal_achievements': [],
    'cancelled': 0
})

for p in all_plans:
    pm = p['project_manager']
    pm_stats[pm]['plans'].append(p)
    
    if p['status'] == 'cancelled':
        pm_stats[pm]['cancelled'] += 1
    elif p['status'] == 'approved':
        if p.get('approved_surplus_pct') is not None and p.get('actual_surplus_pct') is not None:
            variance = abs((p['actual_surplus_pct'] - p['approved_surplus_pct']) * 100)
            pm_stats[pm]['surplus_vars'].append(variance)
        
        if p.get('pct_goal') is not None:
            pm_stats[pm]['goal_achievements'].append(p['pct_goal'])

# Calculate averages per PM
pm_summary = []
for pm, stats in pm_stats.items():
    total = len(stats['plans'])
    if total < 5:  # Skip PMs with too few plans
        continue
    
    pm_summary.append({
        'pm': pm,
        'total_plans': total,
        'avg_surplus_var': statistics.mean(stats['surplus_vars']) if stats['surplus_vars'] else None,
        'avg_goal': statistics.mean(stats['goal_achievements']) if stats['goal_achievements'] else None,
        'cancel_rate': (stats['cancelled'] / total * 100) if total > 0 else 0
    })

# Find mentors for each weakness
target_stats = next(s for s in pm_summary if s['pm'] == target_pm)

if target_stats['avg_surplus_var'] and target_stats['avg_surplus_var'] > avg_surplus_var:
    surplus_mentors = sorted([s for s in pm_summary if s['avg_surplus_var']], 
                            key=lambda x: x['avg_surplus_var'])[:3]
    print("\nSurplus Control Mentors:")
    for m in surplus_mentors:
        print(f"  {m['pm']}: {m['avg_surplus_var']:.1f}% avg variance")

if target_stats['avg_goal'] and target_stats['avg_goal'] < avg_goal:
    goal_mentors = sorted([s for s in pm_summary if s['avg_goal']], 
                         key=lambda x: x['avg_goal'], reverse=True)[:3]
    print("\nGoal Achievement Mentors:")
    for m in goal_mentors:
        print(f"  {m['pm']}: {m['avg_goal']:.1f}% avg achievement")

# Identify who the target PM could mentor
if target_stats['avg_goal'] and target_stats['avg_goal'] > avg_goal:
    mentees = sorted([s for s in pm_summary if s['avg_goal'] and s['avg_goal'] < target_stats['avg_goal']], 
                    key=lambda x: x['avg_goal'])[:3]
    print(f"\n{target_pm} Could Mentor (Goal Achievement):")
    for m in mentees:
        print(f"  {m['pm']}: {m['avg_goal']:.1f}% avg achievement")
```

### 7. Look for patterns in problem areas

Investigate if issues correlate with specific planners, jobs, or time periods:

```python
# Check if cancellations cluster by planner
cancelled_plans = [p for p in target_plans if p['status'] == 'cancelled']
planner_cancellations = defaultdict(int)
for p in cancelled_plans:
    planner_cancellations[p['planner']] += 1

if planner_cancellations:
    print("\nCancellations by Planner:")
    for planner, count in sorted(planner_cancellations.items(), key=lambda x: x[1], reverse=True):
        print(f"  {planner}: {count} cancellations")

# Check if low performers cluster by date
low_performers = [p for p in with_goals if p['pct_goal'] < 80]
if low_performers:
    print("\nLow Performers by Date:")
    for p in sorted(low_performers, key=lambda x: x['start_on']):
        print(f"  {p['start_on']}: Plan {p['id']} at {p['pct_goal']:.1f}%")
```

## Key Metrics to Calculate

1. **Surplus Control**: Average absolute variance between approved and actual surplus percentages (lower is better)
2. **Goal Achievement**: Average percentage of tonnage goals achieved (100%+ is good)
3. **Cancellation Rate**: Percentage of plans cancelled (lower is better)
4. **Large-Scale Management**: Performance on plans >2000 tons
5. **Consistency**: Standard deviation of goal achievement (lower is more consistent)

## Tips

- Always filter to `status == 'approved'` for performance metrics (exclude cancelled/draft)
- Look for patterns in underperformance (specific jobs, planners, time periods)
- Consider both relative (vs peers) and absolute performance
- Identify 3-5 specific plan IDs for each strength/weakness to make feedback concrete
- For mentorship pairing, match complementary strengths (one's strength is another's weakness)
