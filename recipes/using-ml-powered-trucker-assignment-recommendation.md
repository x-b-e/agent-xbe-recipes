---
title: Using ML-powered trucker assignment recommendations for lineup shifts
when: When you need to assign truckers to lineup shifts and want to use the system's ML-powered recommendation engine to identify the best candidates ranked by probability
---

# Using ML-powered trucker assignment recommendations for lineup shifts

## Context
The system provides ML-powered recommendations for assigning truckers to lineup shifts. The recommendation engine analyzes historical patterns, trucker performance, and other factors to rank candidate truckers by probability of being the best match.

## Important: Use Lineup Shift IDs, Not Job Schedule Shift IDs
The ML recommendation system works with **lineup-job-schedule-shifts**, not job-schedule-shifts. You must first create the lineup shift before generating recommendations.

## Workflow

### Step 1: Create Lineup Shift (Without Trucker)
```bash
# Create the lineup shift first
lineup_shift_id=$(xbe do lineup-job-schedule-shifts create \
  --lineup <lineup-id> \
  --job-schedule-shift <job-schedule-shift-id> \
  --is-ready-to-dispatch false \
  --json | jq -r '.id')
```

### Step 2: Generate ML Recommendations
```bash
# Create recommendation for the lineup shift
rec_id=$(xbe do lineup-job-schedule-shift-trucker-assignment-recommendations create \
  --lineup-job-schedule-shift $lineup_shift_id \
  --json | jq -r '.id')
```

### Step 3: View Recommendations
```bash
# View top recommendations with probabilities
xbe view lineup-job-schedule-shift-trucker-assignment-recommendations show $rec_id \
  --fields candidates \
  --json
```

Example output structure:
```json
{
  "candidates": [
    {
      "rank": 0,
      "probability": 0.67,
      "trucker": {
        "id": "1126",
        "company-name": "AB Rock"
      }
    },
    {
      "rank": 1,
      "probability": 0.12,
      "trucker": {
        "id": "2341",
        "company-name": "Scot Heller"
      }
    },
    {
      "rank": 2,
      "probability": 0.08,
      "trucker": {
        "id": "3452",
        "company-name": "Knifong"
      }
    }
  ]
}
```

### Step 4: Assign Chosen Trucker
```bash
# Assign the chosen trucker (e.g., rank 0 recommendation)
xbe do lineup-job-schedule-shifts update $lineup_shift_id \
  --trucker <chosen-trucker-id> \
  --is-ready-to-dispatch true
```

## Extracting Top Recommendation
```bash
# Get the top recommended trucker ID
top_trucker=$(xbe view lineup-job-schedule-shift-trucker-assignment-recommendations show $rec_id \
  --json | jq -r '.candidates | sort_by(.rank) | .[0].trucker.id')

# Auto-assign top recommendation
xbe do lineup-job-schedule-shifts update $lineup_shift_id \
  --trucker $top_trucker \
  --is-ready-to-dispatch true
```

## Formatting for Human Review
```bash
# Show recommendations in readable format
xbe view lineup-job-schedule-shift-trucker-assignment-recommendations show $rec_id \
  --json | jq -r '.candidates | sort_by(.rank) | .[] | "Rank \(.rank): \(.trucker."company-name") (\((.probability * 100)|floor)%)"'
```

Example output:
```
Rank 0: AB Rock (67%)
Rank 1: Scot Heller (12%)
Rank 2: Knifong (8%)
```

## Batch Processing Multiple Shifts
```bash
# Process all shifts in a lineup with ML recommendations
for shift_id in $(xbe view lineup-job-schedule-shifts list \
  --lineup <lineup-id> \
  --json | jq -r '.[] | select(.trucker == null) | .id'); do
  
  # Generate recommendation
  rec_id=$(xbe do lineup-job-schedule-shift-trucker-assignment-recommendations create \
    --lineup-job-schedule-shift $shift_id \
    --json | jq -r '.id')
  
  # Show recommendations for manual review
  echo "Shift $shift_id recommendations:"
  xbe view lineup-job-schedule-shift-trucker-assignment-recommendations show $rec_id \
    --json | jq -r '.candidates | sort_by(.rank) | .[] | "  Rank \(.rank): \(.trucker."company-name") (\((.probability * 100)|floor)%)"'
  
  # Optional: auto-assign top recommendation
  # top_trucker=$(xbe view lineup-job-schedule-shift-trucker-assignment-recommendations show $rec_id \
  #   --json | jq -r '.candidates | sort_by(.rank) | .[0].trucker.id')
  # xbe do lineup-job-schedule-shifts update $shift_id --trucker $top_trucker --is-ready-to-dispatch true
done
```

## Common Issues

### Error: Cannot create recommendation for job-schedule-shift
The ML system requires a **lineup-job-schedule-shift** ID, not a job-schedule-shift ID. You must first add the shift to a lineup:
```bash
# Wrong: Using job-schedule-shift ID
xbe do lineup-job-schedule-shift-trucker-assignment-recommendations create \
  --lineup-job-schedule-shift <job-schedule-shift-id>  # ❌ Wrong resource type

# Right: Create lineup shift first, then use its ID
lineup_shift_id=$(xbe do lineup-job-schedule-shifts create \
  --lineup <lineup-id> \
  --job-schedule-shift <job-schedule-shift-id> \
  --json | jq -r '.id')

xbe do lineup-job-schedule-shift-trucker-assignment-recommendations create \
  --lineup-job-schedule-shift $lineup_shift_id  # ✓ Correct
```

### Understanding the Resource Hierarchy
- `job-schedule-shift`: The original shift that needs to be worked
- `lineup-job-schedule-shift`: That shift as part of a specific lineup (has its own ID)
- `recommendation`: Created for the lineup-job-schedule-shift
- `trucker assignment`: Set on the lineup-job-schedule-shift

## Notes
- Recommendations are based on ML analysis of historical assignments, performance, and patterns
- The probability scores represent the model's confidence in each recommendation
- You can accept or override recommendations based on business knowledge
- Recommendations are specific to the lineup shift context (considering the lineup, job, timing, etc.)
