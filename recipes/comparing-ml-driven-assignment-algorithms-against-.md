---
title: Comparing ML-driven assignment algorithms against actual dispatcher decisions
when: When you need to evaluate how closely an automated assignment algorithm (given a predetermined allocation) can replicate actual dispatcher decisions, understand where and why they diverge, and assess whether the algorithm or dispatcher makes better choices based on ML confidence scores
---

## Overview

This recipe shows how to compare an ML-driven trucker assignment algorithm against actual dispatcher assignments to understand drift patterns, identify where they agree/disagree, and evaluate decision quality.

## Prerequisites

- A lineup ID with dispatched shifts
- ML recommendation data available for the shifts (ml1_id, ml1_prob, ml2_id, ml2_prob, ml3_id, ml3_prob)
- Actual trucker assignments (trucker-id field)

## Step 1: Fetch Lineup Shifts with All Required Data

Get the lineup shifts with ML recommendations and actual assignments:

```bash
xbe view lineup-shifts list \
  --lineup <lineup-id> \
  --fields shift-id,trucker-id,ml1-id,ml1-prob,ml2-id,ml2-prob,ml3-id,ml3-prob \
  --json > shifts.json
```

## Step 2: Extract Actual Allocation

Determine how many shifts each trucker actually received:

```bash
jq -r '.[] | ."trucker-id"' shifts.json | sort | uniq -c | sort -rn
```

This gives you the allocation constraint that both the dispatcher and algorithm must satisfy.

## Step 3: Get Trucker Names for Reporting

```bash
jq -r '.[] | ."trucker-id"' shifts.json | sort -u | while read tid; do
  xbe view truckers show "$tid" --fields company-name --json
done | jq -s 'map({(.id): ."company-name"}) | add' > trucker_names.json
```

## Step 4: Build Assignment Algorithm

Create a Python script that implements an allocation-aware greedy assignment algorithm:

```python
import json
import sys
from collections import defaultdict

# Load data
with open('shifts.json') as f:
    shifts = json.load(f)

with open('trucker_names.json') as f:
    trucker_names = json.load(f)

# Extract actual allocation
actual_allocation = defaultdict(int)
for shift in shifts:
    actual_allocation[shift['trucker-id']] += 1

# Build assignment algorithm (greedy with allocation constraints)
recommended = {}
available_capacity = actual_allocation.copy()

# Helper to get ML probability for a trucker on a shift
def get_ml_prob(shift, trucker_id):
    if shift.get('ml1-id') == trucker_id:
        return shift.get('ml1-prob', 0.0)
    elif shift.get('ml2-id') == trucker_id:
        return shift.get('ml2-prob', 0.0)
    elif shift.get('ml3-id') == trucker_id:
        return shift.get('ml3-prob', 0.0)
    return 0.0

# Sort shifts by ML confidence (assign high-confidence shifts first)
shifts_sorted = sorted(shifts, 
    key=lambda s: max(get_ml_prob(s, tid) for tid in actual_allocation.keys()),
    reverse=True)

# Greedy assignment
for shift in shifts_sorted:
    shift_id = shift['shift-id']
    
    # Find best available trucker based on ML score and remaining capacity
    best_trucker = None
    best_prob = -1
    
    for trucker_id, capacity in available_capacity.items():
        if capacity > 0:
            prob = get_ml_prob(shift, trucker_id)
            if prob > best_prob:
                best_prob = prob
                best_trucker = trucker_id
    
    if best_trucker:
        recommended[shift_id] = best_trucker
        available_capacity[best_trucker] -= 1

# Output recommendations
with open('recommended.json', 'w') as f:
    json.dump(recommended, f)
```

## Step 5: Generate Comprehensive Drift Report

Create a detailed comparison report:

```python
import json
from collections import defaultdict

# Load all data
with open('shifts.json') as f:
    shifts_data = json.load(f)

with open('trucker_names.json') as f:
    trucker_names = json.load(f)

with open('recommended.json') as f:
    recommended = json.load(f)

# Calculate match rate
matches = sum(1 for s in shifts_data 
              if s['trucker-id'] == recommended[s['shift-id']])
total = len(shifts_data)
match_pct = 100 * matches / total

print(f"Overall Match Rate: {matches}/{total} shifts ({match_pct:.1f}%)")
print(f"Drift: {total - matches} shifts ({100 - match_pct:.1f}%)")
print()

# Calculate average ML confidence
def get_ml_prob(shift, trucker_id):
    if shift.get('ml1-id') == trucker_id:
        return shift.get('ml1-prob', 0.0)
    if shift.get('ml2-id') == trucker_id:
        return shift.get('ml2-prob', 0.0)
    if shift.get('ml3-id') == trucker_id:
        return shift.get('ml3-prob', 0.0)
    return 0.0

actual_conf = sum(get_ml_prob(s, s['trucker-id']) for s in shifts_data) / total
algo_conf = sum(get_ml_prob(s, recommended[s['shift-id']]) for s in shifts_data) / total

print(f"Average ML Confidence:")
print(f"  Actual dispatcher:  {actual_conf*100:.1f}%")
print(f"  Algorithm:          {algo_conf*100:.1f}%")
print(f"  Difference:         {(algo_conf - actual_conf)*100:+.1f}%")
print()

# Trucker-by-trucker breakdown
actual_allocation = defaultdict(int)
for shift in shifts_data:
    actual_allocation[shift['trucker-id']] += 1

for trucker_id in sorted(actual_allocation.keys(), 
                         key=lambda t: actual_allocation[t], 
                         reverse=True):
    name = trucker_names.get(trucker_id, f"Unknown ({trucker_id})")
    count = actual_allocation[trucker_id]
    
    actual_shifts = [s for s in shifts_data if s['trucker-id'] == trucker_id]
    rec_shifts = [s for s in shifts_data if recommended[s['shift-id']] == trucker_id]
    
    actual_ids = set(s['shift-id'] for s in actual_shifts)
    rec_ids = set(s['shift-id'] for s in rec_shifts)
    
    overlap = actual_ids & rec_ids
    match_pct = 100 * len(overlap) / count if count > 0 else 0
    
    print(f"{name}: {len(overlap)}/{count} shifts match ({match_pct:.0f}%)")
    
    # Show drifted shifts for this trucker
    only_actual = actual_ids - rec_ids
    if only_actual:
        print(f"  Dispatcher assigned but algorithm would NOT:")
        for shift_id in sorted(only_actual):
            shift = next(s for s in shifts_data if s['shift-id'] == shift_id)
            actual_ml = get_ml_prob(shift, trucker_id) * 100
            rec_tid = recommended[shift_id]
            rec_ml = get_ml_prob(shift, rec_tid) * 100
            print(f"    {shift_id}: {actual_ml:.1f}% → Algorithm chose: {trucker_names.get(rec_tid)} {rec_ml:.1f}%")

print()

# Drift pattern analysis
drifted = [(s, recommended[s['shift-id']]) for s in shifts_data
           if s['trucker-id'] != recommended[s['shift-id']]]

if drifted:
    print(f"DRIFT ANALYSIS: {len(drifted)} shifts differ\n")
    
    # Categorize by ML confidence
    low_conf = [s for s, _ in drifted if get_ml_prob(s, s['trucker-id']) < 0.15]
    med_conf = [s for s, _ in drifted if 0.15 <= get_ml_prob(s, s['trucker-id']) < 0.30]
    high_conf = [s for s, _ in drifted if get_ml_prob(s, s['trucker-id']) >= 0.30]
    
    print(f"Drift by ML confidence:")
    print(f"  Low (<15%):    {len(low_conf)} shifts")
    print(f"  Medium (15-30%): {len(med_conf)} shifts")
    print(f"  High (>30%):    {len(high_conf)} shifts")
    print()
    
    # Who made better choices?
    algo_better = sum(1 for s, rec_tid in drifted
                      if get_ml_prob(s, rec_tid) > get_ml_prob(s, s['trucker-id']))
    dispatcher_better = sum(1 for s, rec_tid in drifted
                            if get_ml_prob(s, s['trucker-id']) > get_ml_prob(s, rec_tid))
    
    print(f"When they differ, who has higher ML confidence?")
    print(f"  Algorithm:   {algo_better} shifts")
    print(f"  Dispatcher:  {dispatcher_better} shifts")
    print(f"  Tied:        {len(drifted) - algo_better - dispatcher_better} shifts")
```

## Key Insights to Look For

1. **Match Rate**: How often does the algorithm replicate dispatcher decisions?
2. **ML Confidence Comparison**: Does the algorithm achieve higher average ML confidence?
3. **Drift Patterns**: Where do they disagree?
   - Low-confidence shifts (neither has strong preference) - usually noise
   - High-confidence shifts (ML has clear preference) - meaningful divergence
4. **Trucker-Specific Patterns**: Do certain truckers show more drift than others?
5. **Quality Assessment**: When they differ, who makes better choices based on ML scores?

## Common Findings

- High match rates (80-90%) indicate the allocation constraint is the primary driver
- Most drift occurs in low-confidence assignments where ML has no strong preference
- Meaningful drift (high-confidence disagreements) reveals where dispatcher uses non-ML factors
- Small truckers with 1-2 shifts often show 0% match, but this is usually arbitrary assignment of leftover shifts
