---
title: Optimizing shift-to-trucker assignments using ML recommendations with fixed allocation
when: When you need to assign lineup shifts to truckers given a predetermined allocation (e.g., 5 shifts to trucker A, 3 to trucker B) and want to maximize the quality of matches using ML recommendation scores
---

# Optimizing shift-to-trucker assignments using ML recommendations with fixed allocation

When you have:
- A predetermined allocation of how many shifts each trucker should receive
- ML recommendations for each shift indicating which truckers are good fits
- Need to decide WHICH specific shifts to give to WHICH specific truckers

This recipe provides an algorithm to maximize overall ML confidence while respecting allocation constraints.

## Problem Statement

**Given:**
- Allocation: `{<trucker-id-1>: <count-1>, <trucker-id-2>: <count-2>, ...}`
- ML Recommendations: `{<shift-id>: [(<trucker-id>, <probability>), ...], ...}`

**Goal:** Assign each shift to exactly one trucker such that:
1. Each trucker gets exactly their allocated number of shifts
2. Overall ML confidence is maximized
3. Strong ML matches get priority

## Algorithm: Sequential Assignment by ML Strength

### Phase 1: Generate ML Recommendations

```bash
# For each unassigned shift in the lineup
for shift_id in $(xbe view lineup-job-schedule-shifts list \
  --lineup <lineup-id> --json | jq -r '.[].id'); do
  
  # Generate recommendations
  rec_id=$(xbe do lineup-job-schedule-shift-trucker-assignment-recommendations create \
    --lineup-job-schedule-shift $shift_id --json | jq -r '.id')
  
  # Get recommendations with probabilities
  xbe view lineup-job-schedule-shift-trucker-assignment-recommendations show $rec_id \
    --fields trucker-assignments --json > /tmp/rec_${shift_id}.json
done
```

### Phase 2: Calculate Trucker ML Strength

```python
# For each trucker, sum their ML probabilities across all shifts where they appear in top-3
trucker_strength = {}
for trucker_id in allocation.keys():
    total_prob = 0
    for shift_id, recommendations in all_recommendations.items():
        for rec_trucker_id, probability in recommendations[:3]:  # top-3 only
            if rec_trucker_id == trucker_id:
                total_prob += probability
                break
    trucker_strength[trucker_id] = total_prob

# Sort truckers by ML strength (strongest first)
trucker_order = sorted(allocation.keys(), 
                       key=lambda t: trucker_strength[t], 
                       reverse=True)
```

**Why this matters:** Truckers with strong ML matches across many shifts get to pick first, ensuring the best fits aren't left over.

### Phase 3: Sequential Assignment (Company-by-Company)

```python
assignments = {}
unassigned_shifts = set(all_shift_ids)

# Process truckers in ML-strength order
for trucker_id in trucker_order:
    count_needed = allocation[trucker_id]
    
    # Score all remaining unassigned shifts for THIS trucker
    candidates = []
    for shift_id in unassigned_shifts:
        ml_score = get_ml_probability(shift_id, trucker_id)
        candidates.append((shift_id, ml_score))
    
    # Sort by ML probability (best fits first)
    candidates.sort(key=lambda x: x[1], reverse=True)
    
    # Assign top N shifts to this trucker
    for shift_id, score in candidates[:count_needed]:
        assignments[shift_id] = trucker_id
        unassigned_shifts.remove(shift_id)
```

### Phase 4: Execute Assignments

```bash
# Apply the computed assignments
for shift_id in "${!assignments[@]}"; do
  trucker_id="${assignments[$shift_id]}"
  
  xbe do lineup-job-schedule-shifts update $shift_id \
    --trucker $trucker_id \
    --is-ready-to-dispatch true
done
```

## Example Workflow

```bash
# Dispatcher determines allocation strategy
allocation={
  "<trucker-id-1>": 5,  # Scot Heller
  "<trucker-id-2>": 5,  # Knifong
  "<trucker-id-3>": 3,  # Velox
  "<trucker-id-4>": 3,  # KDB Transport
  "<trucker-id-5>": 2,  # Martin Rucker
  "<trucker-id-6>": 1   # AB Rock
}

# System runs matching algorithm:
# 1. Scot Heller (ML strength: 604) → Takes 5 best shifts (avg 51% confidence)
# 2. Martin Rucker (ML strength: 230) → Takes 2 best remaining (avg 18%)
# 3. Velox (ML strength: 132) → Takes 3 best remaining (avg 18%)
# 4. Knifong (ML strength: 112) → Takes 5 best remaining (avg 14%)
# 5. KDB (ML strength: 76) → Takes 3 remaining (avg 0% - leftovers)
# 6. AB Rock (ML strength: 67) → Takes 1 remaining (avg 0%)

# Result metrics:
# - 79% of shifts assigned to top-3 ML recommendations
# - 22% overall average ML confidence
# - All allocation constraints satisfied
```

## Why This Algorithm Works

1. **Respects allocation:** Each trucker gets exactly their predetermined count
2. **Maximizes ML confidence:** Within each allocation, picks highest-probability shifts
3. **First-mover advantage:** Truckers with strong ML affinity get first pick of the pool
4. **Company-by-company:** Matches dispatcher mental model ("assign all of Scot Heller's shifts, then move to next company")
5. **No manual decisions:** Automatic optimization within human-determined strategy

## Comparison to Naive Approaches

**Naive approach 1 (global greedy):**
- Assigns shift-by-shift to highest ML probability globally
- Problem: May violate allocation constraints

**Naive approach 2 (random assignment):**
- Randomly assigns shifts respecting counts
- Problem: Ignores ML recommendations entirely

**This sophisticated approach:**
- Strategic allocation order (strong ML matches first)
- Local optimization (best fits within each allocation)
- Mimics sophisticated dispatcher thinking: "Given Scot Heller needs 5 shifts, which 5 are their best fits?"

## Implementation Notes

- The algorithm can be implemented in Python (as shown) or bash with jq
- ML strength calculation focuses on top-3 recommendations to avoid noise
- Sequential processing ensures no conflicts or double-assignments
- The predetermined allocation can come from:
  - Dispatcher's business judgment
  - Historical patterns
  - Capacity constraints
  - Customer preferences
