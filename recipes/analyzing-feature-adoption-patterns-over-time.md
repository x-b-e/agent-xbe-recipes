---
title: Analyzing feature adoption patterns over time
when: When you need to analyze how a feature (like managed transport) is being adopted across different offices/locations over time, including calculating adoption velocity, identifying patterns, and forecasting future adoption
---

# Analyzing feature adoption patterns over time

## Overview
This recipe shows how to analyze feature adoption (e.g., managed transport orders) across multiple offices over time, calculate adoption velocity, identify patterns, and create data-backed forecasts.

## Step 1: Find the broker ID
```bash
xbe view brokers list --q "<broker-name>" --json
```

## Step 2: Fetch historical data with appropriate date range
```bash
xbe view transport-orders list \
  --broker <broker-id> \
  --start-on <start-date> \
  --end-on <end-date> \
  --json > /tmp/transport_orders.json
```

**Note:** For multi-month analyses, you may need to fetch data in chunks if the date range is too large.

## Step 3: Calculate weekly adoption percentages by office

Use `jq` to group orders by week and office, then calculate managed percentages:

```bash
jq -r '
# Group by ISO week and office
group_by(.pickup_at[:10] | strptime("%Y-%m-%d") | strftime("%Y-W%U"), .project_office_name) |
map({
  week: .[0].pickup_at[:10] | strptime("%Y-%m-%d") | strftime("%Y-W%U"),
  office: .[0].project_office_name,
  total: length,
  managed: map(select(.is_managed == true)) | length
}) |
map(. + {pct_managed: ((.managed / .total) * 100 | round)}) |
.[] | "\(.week),\(.office),\(.total),\(.managed),\(.pct_managed)"
' /tmp/transport_orders.json > /tmp/weekly_adoption.csv
```

## Step 4: Analyze adoption velocity patterns

Create a script to calculate time-to-adoption milestones:

```bash
cat > /tmp/analyze_adoption.py << 'EOF'
import csv
from collections import defaultdict
from datetime import datetime

# Read weekly data
data = defaultdict(list)
with open('/tmp/weekly_adoption.csv') as f:
    for line in f:
        week, office, total, managed, pct = line.strip().split(',')
        data[office].append({
            'week': week,
            'total': int(total),
            'managed': int(managed),
            'pct': int(pct)
        })

# Calculate adoption velocity for each office
for office, weeks in data.items():
    # Find first week >5% (adoption start)
    start_week = next((w['week'] for w in weeks if w['pct'] > 5), None)
    # Find first week >50%
    week_50 = next((w['week'] for w in weeks if w['pct'] >= 50), None)
    # Find first week >80%
    week_80 = next((w['week'] for w in weeks if w['pct'] >= 80), None)
    
    if start_week and week_80:
        # Calculate weeks to reach milestones
        print(f"{office}: {start_week} -> 80% in {week_80}")
EOF

python3 /tmp/analyze_adoption.py
```

## Step 5: Create visualization and report

### Option A: Quick ASCII visualization
```bash
cat /tmp/weekly_adoption.csv | \
  awk -F',' '{print $2, $5}' | \
  sort | uniq |
  # Add ASCII bar chart logic here
```

### Option B: Detailed markdown report

Create a comprehensive analysis document:

```bash
cat > /tmp/adoption_report.md << 'EOF'
# Feature Adoption Analysis Report

## Executive Summary
[Summarize key findings: median adoption time, patterns observed, forecasts]

## Pattern Analysis

### Adoption Velocity Breakdown
- **Rapid adoption** (<=2 weeks to 80%): [list offices]
- **Moderate adoption** (3-4 weeks): [list offices]
- **Gradual adoption** (5-6 weeks): [list offices]

### Key Insights
1. **Adoption pattern**: [Describe whether adoption is gradual or "flip the switch"]
2. **Learning curve effect**: [Compare early vs late adopters]
3. **Timeline feasibility**: [Data-backed projection]

## Recommendations
[Based on patterns observed]
EOF
```

## Step 6: Create concise executive communication

For stakeholder updates, create a focused email:

```bash
cat > /tmp/adoption_email.txt << 'EOF'
Subject: [Concise summary of finding]

THE DATA:
- Current state: [key metrics]
- Proven patterns: [evidence from analysis]
- Timeline: [projection with supporting data]

THE PATH FORWARD:
[Specific action items based on data]

BOTTOM LINE:
[One-sentence key takeaway]
EOF
```

## Key Insights from This Analysis Pattern

1. **Adoption is often binary, not gradual**: Offices tend to "flip the switch" rather than gradually ramp up
2. **Later adopters are faster**: Learning curve effects mean knowledge transfer accelerates adoption
3. **Use milestones**: Track time to 50% and 80% adoption, not just current percentages
4. **Compare cohorts**: Early vs late adopters reveal process improvement trends

## Tips

- **Date ranges**: For historical analysis spanning months, consider fetching in chunks
- **Week numbering**: ISO weeks (W01-W53) are useful for time-series analysis
- **Threshold selection**: 5% for "adoption started", 50% for "halfway", 80% for "mostly adopted"
- **Outlier analysis**: Identify both fast adopters (to replicate success) and slow/stalled offices (to diagnose blockers)
