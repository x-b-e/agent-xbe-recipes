---
title: Comparing ML recommendations to actual dispatcher assignments with probability analysis
when: When you need to evaluate how well dispatcher assignments align with ML recommendations, analyze the quality of both choices based on ML confidence scores, or understand whether an algorithm would perform better than human dispatchers
---

## Overview

This recipe shows how to compare ML-recommended trucker assignments against actual dispatcher choices, focusing on ML probability scores to evaluate decision quality.

## Key Concepts

**ML Recommendations**: Each lineup shift has ML-ranked candidate truckers with probability scores
**Actual Assignments**: What the dispatcher actually chose
**Probability-Based Evaluation**: Comparing choices based on ML confidence rather than simple match/mismatch

## Step-by-Step Process

### 1. Get Lineup Shifts with ML Data

First, retrieve shifts for the lineup you want to analyze:

```bash
xbe view lineup-shifts list \
  --lineup <lineup-id> \
  --fields trucker-id,ml-trucker-candidates \
  --json > shifts.json
```

The `ml-trucker-candidates` field contains the ranked list of ML recommendations with probabilities.

### 2. Create Analysis Script

Create a Python script to compare assignments and analyze probability distributions:

```python
import json
from collections import defaultdict

# Load data
with open('shifts.json') as f:
    shifts = json.load(f)

# Load trucker names for reporting
with open('truckers.json') as f:
    truckers = {t['id']: t['company-name'] for t in json.load(f)}

# Analysis structures
data = []
for shift in shifts:
    shift_id = shift['id']
    actual_trucker_id = shift['trucker-id']
    ml_candidates = shift['ml-trucker-candidates']
    
    # Find actual trucker in ML ranking
    actual_rank = None
    actual_prob = 0
    for idx, candidate in enumerate(ml_candidates):
        if candidate['trucker-id'] == actual_trucker_id:
            actual_rank = idx + 1
            actual_prob = candidate['probability']
            break
    
    # Get top ML recommendation
    top_ml = ml_candidates[0]
    
    data.append({
        'shift_id': shift_id,
        'actual_trucker_id': actual_trucker_id,
        'actual_name': truckers[actual_trucker_id],
        'actual_prob': actual_prob,
        'actual_rank': actual_rank,
        'ml_trucker_id': top_ml['trucker-id'],
        'ml_name': truckers[top_ml['trucker-id']],
        'ml_prob': top_ml['probability'],
        'matches': actual_trucker_id == top_ml['trucker-id'],
        'delta': top_ml['probability'] - actual_prob
    })

# Calculate statistics
total_shifts = len(data)
matches = sum(1 for d in data if d['matches'])
avg_actual_prob = sum(d['actual_prob'] for d in data) / total_shifts
avg_ml_prob = sum(d['ml_prob'] for d in data) / total_shifts

print(f"Overall Match Rate: {matches}/{total_shifts} ({matches*100/total_shifts:.1f}%)")
print(f"Average ML Confidence:")
print(f"  Dispatcher choices: {avg_actual_prob*100:.2f}%")
print(f"  ML top picks: {avg_ml_prob*100:.2f}%")
print(f"  Difference: {(avg_ml_prob - avg_actual_prob)*100:+.2f}%")

# Categorize by probability difference
large_improvement = [d for d in data if d['delta'] > 0.10]  # >10% better
small_improvement = [d for d in data if 0 < d['delta'] <= 0.10]
neutral = [d for d in data if d['delta'] == 0]
small_worse = [d for d in data if -0.10 <= d['delta'] < 0]
large_worse = [d for d in data if d['delta'] < -0.10]  # >10% worse

print(f"\nImpact Distribution:")
print(f"  Large improvement (>10%): {len(large_improvement)} ({len(large_improvement)*100/total_shifts:.1f}%)")
print(f"  Small improvement (0-10%): {len(small_improvement)} ({len(small_improvement)*100/total_shifts:.1f}%)")
print(f"  Neutral (same): {len(neutral)} ({len(neutral)*100/total_shifts:.1f}%)")
print(f"  Small worse (0-10%): {len(small_worse)} ({len(small_worse)*100/total_shifts:.1f}%)")
print(f"  Large worse (>10%): {len(large_worse)} ({len(large_worse)*100/total_shifts:.1f}%)")

# Show examples of large differences
if large_improvement:
    print(f"\nLarge improvements (ML significantly better):")
    for item in sorted(large_improvement, key=lambda x: x['delta'], reverse=True)[:5]:
        print(f"  Shift {item['shift_id']}: Dispatcher={item['actual_name']} ({item['actual_prob']*100:.1f}%, rank #{item['actual_rank']}), ML={item['ml_name']} ({item['ml_prob']*100:.1f}%)")

if large_worse:
    print(f"\nLarge regressions (Dispatcher significantly better):")
    for item in sorted(large_worse, key=lambda x: x['delta'])[:5]:
        print(f"  Shift {item['shift_id']}: Dispatcher={item['actual_name']} ({item['actual_prob']*100:.1f}%, rank #1), ML={item['ml_name']} ({item['ml_prob']*100:.1f}%)")
```

### 3. Analyze Probability Distributions

Add distribution analysis to understand confidence patterns:

```python
# Categorize by confidence level
def categorize_confidence(prob):
    if prob >= 0.50: return 'Very High (>=50%)'
    if prob >= 0.30: return 'High (30-50%)'
    if prob >= 0.15: return 'Medium (15-30%)'
    if prob >= 0.05: return 'Low (5-15%)'
    return 'Very Low (<5%)'

actual_dist = defaultdict(int)
ml_dist = defaultdict(int)

for d in data:
    actual_dist[categorize_confidence(d['actual_prob'])] += 1
    ml_dist[categorize_confidence(d['ml_prob'])] += 1

print("\nConfidence Distribution:")
print(f"{'Category':<20} {'Dispatcher':>12} {'ML Top Pick':>12}")
for category in ['Very High (>=50%)', 'High (30-50%)', 'Medium (15-30%)', 'Low (5-15%)', 'Very Low (<5%)']:
    print(f"{category:<20} {actual_dist[category]:>12} {ml_dist[category]:>12}")
```

### 4. Create Detailed Shift-by-Shift Report

Generate a sortable report showing every shift:

```python
print("\nShift-by-Shift Comparison (sorted by dispatcher ML confidence):")
print(f"{'Shift':<10} {'Match':>5} {'Dispatcher':<30} {'ML%':>5} {'Rank':>4} {'ML Top Pick':<30} {'ML%':>5} {'Delta':>6}")
print("-" * 110)

for d in sorted(data, key=lambda x: x['actual_prob'], reverse=True):
    match_symbol = '✓' if d['matches'] else ' '
    delta_str = f"{d['delta']*100:+.1f}%" if not d['matches'] else '='
    
    print(f"{d['shift_id']:<10} {match_symbol:>5} {d['actual_name'][:30]:<30} {d['actual_prob']*100:>5.1f} #{d['actual_rank']:>3} "
          f"{d['ml_name'][:30]:<30} {d['ml_prob']*100:>5.1f} {delta_str:>6}")
```

## Key Insights from Probability Analysis

**Overall Performance**: Compare average ML confidence between dispatcher choices and ML top picks. A small difference (< 1%) indicates comparable performance.

**Distribution Shifts**: Look for patterns like ML recommendations concentrating more choices in "Very High" confidence vs spreading across categories.

**Trade-off Patterns**: Large improvements in some shifts may be offset by large regressions in others, revealing resource allocation constraints (e.g., limited availability of high-confidence truckers).

**Trucker-Level Patterns**: Analyze average ML confidence by trucker to identify which suppliers are consistently high-confidence vs low-confidence assignments.

## Important Notes

- **Match rate alone is misleading**: 50% match rate with high ML confidence on both choices is very different from 50% with low confidence
- **Resource constraints matter**: ML recommendations may be optimal per-shift but ignore cross-shift constraints (trucker availability, workload balancing)
- **Processing order affects outcomes**: Greedy algorithms are sensitive to the order shifts are processed when high-confidence options are limited
- **Confidence thresholds**: Differences < 5-10% are often not meaningful; focus on large deltas

## Related Recipes

- using-ml-powered-trucker-assignment-recommendation.md - How to access ML recommendations
- tracing-lineup-shift-batch-creation-and-dispatch-t.md - Understanding shift creation patterns
- analyzing-dispatcher-workflow-to-understand-job-se.md - Understanding dispatcher decision-making
