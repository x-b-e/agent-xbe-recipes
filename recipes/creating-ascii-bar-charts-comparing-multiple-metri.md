---
title: Creating ASCII bar charts comparing multiple metrics
when: When you need to create side-by-side ASCII bar chart comparisons of different metrics (like tonnage vs cost per ton) for the same set of entities to visualize relationships or lack thereof
---

# Creating ASCII bar charts comparing multiple metrics

When analyzing lane data, you may want to compare multiple metrics side-by-side to identify correlations or patterns.

## Basic approach

Create a text file with multiple chart sections, each showing a different metric for the same set of lanes:

```bash
cat > /tmp/multi_metric_chart.txt << 'EOF'
================================================================================
           METRIC A vs METRIC B - TOP 10 LANES
================================================================================

METRIC A (units)
--------------------------------------------------------------------------------
Lane 1    value1  ████████████████████████████████████████
Lane 2    value2  ████████████████████████████
Lane 3    value3  ██████████████████

METRIC B (units)
--------------------------------------------------------------------------------
Lane 1    value1  ██████████████████
Lane 2    value2  ████████████████████████████████████████
Lane 3    value3  ████████████████████████

================================================================================
KEY INSIGHTS
================================================================================
• Insight about correlation or lack thereof
• Specific findings about outliers
================================================================================
EOF

cat /tmp/multi_metric_chart.txt
```

## Scaling bars for visual comparison

For each metric section, scale bars independently to use the full width:

```bash
# In awk, find max value for each metric and scale to 40 characters
awk '{
    max_metric_a = find_max_in_column(metric_a_values)
    bar_len = int(current_value / max_metric_a * 40)
}'
```

## Including insights section

Always include a KEY INSIGHTS section that:
- Identifies correlations or lack thereof between metrics
- Highlights outliers (high on one metric, low on another)
- Suggests business implications

## Example from transcript

Comparing tonnage vs cost per ton revealed no volume discount pattern - lanes with highest volume didn't have lowest costs, suggesting pricing is driven by lane characteristics rather than volume leverage.
