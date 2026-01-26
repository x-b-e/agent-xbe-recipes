---
title: Comparing metrics across multiple years in tabular format
when: When you need to compare the same metric (tonnage, transactions, etc.) across multiple years in a side-by-side table format, showing month-by-month and yearly totals
---

# Comparing metrics across multiple years in tabular format

## Problem
You need to compare sales, tonnage, or other metrics across multiple years to identify trends, see year-over-year changes, and understand multi-year patterns.

## Solution
Create a bash script that loops through months and years, querying each combination and formatting as a table:

```bash
echo "Metric Comparison - <start-year>-<end-year>"
echo "============================================================="
echo ""

# Create header
printf "%-10s" "Month"
for year in {<start-year>..<end-year>}; do
  printf " %12s" "$year"
done
echo ""
echo "--------------------------------------------------------------------------------"

# Query each month across all years
for month in {1..12}; do
  month_name=$(date -j -f "%m" "$month" "+%B")
  printf "%-10s" "$month_name"
  
  for year in {<start-year>..<end-year>}; do
    result=$(xbe summarize material-transaction-summary create \
      --group-by "" \
      --filter broker=<broker-id> \
      --filter year=$year \
      --filter month=$month \
      --filter "<additional-filters>" \
      --json 2>/dev/null)
    
    # Extract the metric you want (tons_sum, material_transaction_count, etc.)
    value=$(echo "$result" | jq -r '.rows[0].<metric-field> // 0' 2>/dev/null || echo "0")
    printf " %12.0f" "$value"
  done
  echo ""
done

# Calculate and display yearly totals
echo "--------------------------------------------------------------------------------"
printf "%-10s" "TOTAL"

for year in {<start-year>..<end-year>}; do
  result=$(xbe summarize material-transaction-summary create \
    --group-by "" \
    --filter broker=<broker-id> \
    --filter year=$year \
    --filter "<additional-filters>" \
    --json 2>/dev/null)
  
  value=$(echo "$result" | jq -r '.rows[0].<metric-field> // 0' 2>/dev/null || echo "0")
  printf " %12.0f" "$value"
done
echo ""
```

## Key considerations

### Common metric fields
- `tons_sum` - total tonnage
- `material_transaction_count` - number of transactions
- `cost_sum` - total cost
- `revenue_sum` - total revenue

### Timeout handling
For large date ranges, set a higher timeout:
```bash
xbe summarize material-transaction-summary create \
  --group-by "" \
  --filter ... \
  --json \
  --timeout 300000  # 5 minutes
```

Or run the entire script in the background:
```bash
# Save the script above to a file and run
bash script.sh &
# Monitor with tail -f on the output file
```

### Error handling
The script uses `2>/dev/null` to suppress errors and `// 0` in jq to default to 0 if data is missing, ensuring clean output even with incomplete data.

## Example output
```
Asphalt Sales Comparison - 2020-2025
=============================================================

Month              2020         2021         2022         2023         2024         2025
--------------------------------------------------------------------------------
January           17637         3006         5160        24746         3173         1746
February           2375          776         5237        10830        19658         7629
...
--------------------------------------------------------------------------------
TOTAL           1579659      1640533      1523281      1806620      1381670      1733091
```

## Variations

### Using lane-summary instead
```bash
result=$(xbe summarize lane-summary create \
  --group-by "" \
  --filter broker=<broker-id> \
  --start-on $year-$(printf "%02d" $month)-01 \
  --end-on $year-$(printf "%02d" $month)-31 \
  --json 2>/dev/null)

value=$(echo "$result" | jq -r '.rows[0].tons_sum // 0')
```

### Comparing quarters instead of months
```bash
for quarter in {1..4}; do
  start_month=$((($quarter - 1) * 3 + 1))
  end_month=$(($quarter * 3))
  # Query with date range covering the quarter
done
```
