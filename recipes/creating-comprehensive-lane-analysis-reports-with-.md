---
title: Creating comprehensive lane analysis reports with multiple metrics
when: When you need to analyze lane performance with multiple dimensions including volume, trucker breakdown, costs, and comparative metrics in a formatted report
---

# Creating comprehensive lane analysis reports with multiple metrics

## Context

When analyzing lane performance for a broker, you often need to combine multiple metrics (tonnage, trucker breakdown, cost per ton, hourly rates) into a single comprehensive report that provides both high-level summaries and detailed breakdowns.

## Pattern

Use a single `xbe summarize lane-summary create` command with multiple metrics and JSON output, then process with `jq` and `awk` to create formatted reports:

```bash
xbe summarize lane-summary create \
  --group-by origin,destination \
  --filter broker=<broker-id> \
  --filter year=<year> \
  --metrics tons_sum,effective_rate_per_hour_avg,cost_per_ton,truckers \
  --sort tons_sum:desc \
  --limit <limit> \
  --json | jq -r '.rows[] | ...' | awk '...'
```

## Key Metrics to Include

- `tons_sum` - Total tonnage for volume analysis
- `effective_rate_per_hour_avg` - Profitability metric
- `cost_per_ton` - Unit cost metric
- `truckers` - JSON field containing trucker breakdown with tonnage per trucker

## Processing Trucker Breakdowns

The `truckers` field returns JSON like:
```json
[{"trucker_id": 1, "trucker_name": "ABC Trucking", "tons": 50000}, ...]
```

To extract and format trucker data:

```bash
echo '<truckers-json>' | jq -r '.[] | "\(.trucker_name): \(.tons) tons"'
```

To calculate market share:

```bash
# In awk, with total_tons available:
printf "%.0f%%", (trucker_tons / total_tons * 100)
```

## Report Sections to Include

1. **Lane-by-lane breakdown** with key metrics
2. **Trucker market share** for each lane showing top 3 truckers
3. **Summary statistics** including:
   - Total volume across analyzed lanes
   - Average cost per ton
   - Average hourly rate
   - Highest/lowest rates
   - Most/least balanced lanes

## Example Output Structure

```
================================================================================
LANE ANALYSIS - <BROKER-NAME>
================================================================================

Lane 1: <origin> → <destination>
  Total Volume:     <tons> tons
  Cost per Ton:     $<cost>
  Hourly Rate:      $<rate>/hr
  Top Truckers:
    1. <trucker-name>   <tons> tons  (<percent>%)
    2. <trucker-name>   <tons> tons  (<percent>%)
    3. <trucker-name>   <tons> tons  (<percent>%)

[... repeat for each lane ...]

================================================================================
SUMMARY STATISTICS
================================================================================
Total Volume (Top <N> Lanes):  <tons> tons
Average Cost per Ton:          $<cost>
Average Hourly Rate:           $<rate>
...
================================================================================
```

## Tips

- Use `--limit` to focus on top lanes by volume
- Sort by `tons_sum:desc` to get highest volume lanes first
- Process trucker data to show only top 3 per lane for readability
- Calculate percentages to show market concentration
- Include both absolute values (tons, dollars) and derived metrics (percentages, averages)
