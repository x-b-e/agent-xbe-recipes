---
title: Creating multi-panel lane analysis visualizations with Python
when: When you need to create comprehensive multi-panel visualizations showing lane performance metrics (tonnage, cost per ton, hourly rates) along with trucker breakdowns for presentation or analysis
---

# Creating multi-panel lane analysis visualizations with Python

This recipe shows how to create comprehensive multi-panel visualizations that combine lane summary data with trucker-level breakdowns.

## Prerequisites

1. Get lane summary data with grouping by origin, destination, and trucker:
```bash
xbe view lane-summaries list \
  --broker <broker-id> \
  --from <start-date> \
  --to <end-date> \
  --group origin_id,origin_name,destination_id,destination_name,trucker_id,trucker_name \
  --include effective_cost_per_hour \
  --json > lanes.json
```

2. Ensure Python with matplotlib, pandas, and numpy is available

## Python visualization script structure

The script should:

1. **Aggregate data by lane** (origin-destination pairs) to get total tonnage and weighted averages for cost/hour metrics
2. **Sort lanes** by total tonnage descending and take top N (e.g., 10)
3. **Create multi-panel figure** with:
   - Top panel: Bar chart of total tonnage per lane
   - Middle left: Cost per ton comparison across lanes
   - Middle right: Effective hourly rate comparison across lanes
   - Bottom: Table showing top 3 truckers per lane with their tonnage

## Key implementation patterns

### Aggregating lane data from trucker-level records

```python
import pandas as pd
import numpy as np

df = pd.read_json('lanes.json')

# Create lane identifier
df['lane'] = df['origin_name'] + ' → ' + df['destination_name']

# Group by lane and aggregate
lane_agg = df.groupby('lane').agg({
    'tons': 'sum',
    'total_cost': 'sum',
    'total_cycles': 'sum',
    'total_cycle_hours': 'sum'
}).reset_index()

# Calculate weighted averages
lane_agg['cost_per_ton'] = lane_agg['total_cost'] / lane_agg['tons']
lane_agg['effective_hourly_rate'] = lane_agg['total_cost'] / lane_agg['total_cycle_hours']

# Sort and take top N
top_lanes = lane_agg.nlargest(10, 'tons')
```

### Creating the multi-panel layout

```python
import matplotlib.pyplot as plt
import matplotlib.gridspec as gridspec

fig = plt.figure(figsize=(16, 12))
gs = gridspec.GridSpec(3, 2, height_ratios=[1.5, 1, 1.2], hspace=0.4)

# Top panel: Full width tonnage chart
ax1 = fig.add_subplot(gs[0, :])

# Middle panels: Cost metrics side by side
ax2 = fig.add_subplot(gs[1, 0])
ax3 = fig.add_subplot(gs[1, 1])

# Bottom panel: Full width table
ax4 = fig.add_subplot(gs[2, :])
ax4.axis('off')  # Table doesn't need axes
```

### Building the trucker breakdown table

```python
# For each top lane, find top 3 truckers
table_data = []
for lane in top_lanes['lane']:
    lane_truckers = df[df['lane'] == lane].nlargest(3, 'tons')
    truckers_text = '\n'.join([
        f"{row['trucker_name']}: {row['tons']:,.0f}t"
        for _, row in lane_truckers.iterrows()
    ])
    table_data.append([lane, truckers_text])

# Create table
table = ax4.table(
    cellText=table_data,
    colLabels=['Lane', 'Top 3 Truckers (Tonnage)'],
    cellLoc='left',
    loc='center',
    bbox=[0, 0, 1, 1]
)
table.auto_set_font_size(False)
table.set_fontsize(9)
```

### Formatting currency and rates

```python
# For cost per ton labels
for i, (lane, cost) in enumerate(zip(top_lanes['lane'], top_lanes['cost_per_ton'])):
    ax2.text(cost + 0.1, i, f'${cost:.2f}', va='center')

# For hourly rate labels
for i, (lane, rate) in enumerate(zip(top_lanes['lane'], top_lanes['effective_hourly_rate'])):
    ax3.text(rate + 2, i, f'${rate:.2f}/hr', va='center')
```

## Complete workflow

1. Export lane summary data with trucker grouping and effective cost metrics
2. Write Python script to:
   - Load and aggregate data by lane
   - Identify top N lanes by tonnage
   - Create multi-panel visualization
   - Add trucker breakdown table
3. Run script and save output: `python visualize_lanes.py`
4. Review generated PNG/PDF file

## Key insights to extract

- **Volume leaders**: Which lanes move the most tonnage
- **Cost efficiency**: Which lanes have lowest cost per ton
- **Rate opportunities**: Which lanes command highest hourly rates
- **Trucker concentration**: Whether lanes are served by one dominant trucker or multiple competitors
- **Performance outliers**: Lanes with unusual cost or rate profiles that merit investigation
