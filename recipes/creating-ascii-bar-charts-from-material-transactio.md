---
title: Creating ASCII bar charts from material transaction summaries
when: When you need to visualize material transaction data (tonnage over time) as a quick ASCII bar chart in the terminal
---

# Creating ASCII bar charts from material transaction summaries

## Pattern

Use the material-transaction-summary command with `--json` output, then pipe through `jq` and `awk` to create ASCII visualizations.

## Command Structure

```bash
xbe summarize material-transaction-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --filter material_type_fully_qualified_name_base="<material-category>" \
  --group-by year,month \
  --metrics tons_sum \
  --limit <limit> \
  --json | \
  jq -r '.rows[] | "\(.year)-\(if .month < 10 then "0" else "" end)\(.month)|\(.tons_sum)"' | \
  sort | \
  awk -F'|' 'BEGIN {max=0} {tons[NR]=$2; month[NR]=$1; if($2>max) max=$2} END {for(i=1; i<=NR; i++) {printf "%-8s %9.0f ", month[i], tons[i]; bar_len=int(tons[i]/max*50); for(j=0; j<bar_len; j++) printf "█"; printf "\n"}}'
```

## Pipeline Breakdown

1. **Get JSON data**: Use `--json` flag on the summary command
2. **Format with jq**: Extract year, month, and metric values in pipe-delimited format
   - Zero-pad single-digit months for proper sorting
3. **Sort chronologically**: `sort` ensures data is in time order
4. **Create chart with awk**:
   - Find max value for scaling
   - Format each row with label, value, and proportional bar
   - Use block character `█` for bars

## Example Output

```
2023-01     24746 ████
2023-02     10830 ██
2023-03     49565 █████████
2023-04    214608 █████████████████████████████████████████
2023-05    171350 ████████████████████████████████
```

## Notes

- Chart scales automatically to terminal width (50 characters for bars)
- Works with any metric from material-transaction-summary (tons_sum, cycles_sum, etc.)
- Can be adapted for different groupings by adjusting the jq expression
