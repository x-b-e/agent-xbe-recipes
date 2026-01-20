---
title: Creating ASCII bar charts from lane summaries
when: When you need to visualize lane summary data as a quick ASCII bar chart in the terminal
---

# Creating ASCII bar charts from lane summaries

You can create ASCII bar charts directly from lane summary data by piping the JSON output through jq and awk.

## Basic Pattern

```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter trucker=<trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by origin,destination \
  --metrics tons_sum \
  --sort tons_sum:desc \
  --limit <number> \
  --json | \
jq -r '.rows[] | "\(.origin_name) → \(.destination_name)|\(.tons_sum)"' | \
awk -F'|' 'BEGIN {max=0} {tons[NR]=$2; lane[NR]=$1; if($2>max) max=$2} END {for(i=1; i<=NR; i++) {printf "%-50s %8.0f ", lane[i], tons[i]; bar_len=int(tons[i]/max*50); for(j=0; j<bar_len; j++) printf "█"; printf "\n"}}'
```

## How It Works

1. **xbe command**: Fetches lane summary data in JSON format
2. **jq**: Extracts and formats the data as "lane_name|value" pairs
3. **awk**: 
   - Finds the maximum value to scale bars
   - Prints each lane name (left-aligned, 50 chars)
   - Prints the numeric value (right-aligned, 8 digits)
   - Prints a bar of █ characters proportional to the value

## Customization

- Adjust `%-50s` to change lane name column width
- Adjust `%8.0f` to change number formatting
- Adjust `*50` in `bar_len=int(tons[i]/max*50)` to change maximum bar length
- Change the metric from `tons_sum` to any other available metric
- Use different grouping dimensions (trucker, customer, date, etc.)

## Example Output

```
Randolph → Kansas City-02                           86763 ██████████████████████████████████████████████████
Holliday Shawnee #2/3 → Olathe-07                   78287 █████████████████████████████████████████████
Randolph → Liberty-03                               51937 █████████████████████████████
```
