---
title: Validating data completeness before investigating trucker cost anomalies
when: When you need to investigate why a trucker's cost per ton appears abnormally high and want to rule out missing data before analyzing root causes
---

# Validating Data Completeness Before Investigating Trucker Cost Anomalies

When investigating trucking cost anomalies (e.g., a trucker charging $50/ton vs market rate of $20/ton), the first step is to validate data completeness to ensure the anomaly is real and not due to missing material transaction records.

## Step 1: Get shift summary data

First, retrieve shift-level data to see what the trucker billed:

```bash
xbe view lane-summaries \
  --start-on-min <start-date> \
  --start-on-max <end-date> \
  --broker <broker-id> \
  --trucker <trucker-id> \
  --group-by month,trucker \
  --json > shifts_data.json
```

This gives you:
- Total shifts billed
- Total cost
- Tons recorded on shifts
- Average cost per ton

## Step 2: Get material transaction data

Next, retrieve material transaction records to verify deliveries:

```bash
xbe view material-transactions list \
  --start-on-min <start-date> \
  --start-on-max <end-date> \
  --trucker <trucker-id> \
  --customer <customer-id> \
  --json > material_transactions.json
```

## Step 3: Compare shift tons vs material tons

Use `jq` to aggregate and compare the two datasets:

```bash
# Aggregate shift data by month
cat shifts_data.json | jq -r '
  group_by(.start_on_month) | 
  map({
    month: .[0].start_on_month,
    shift_tons: (map(.tons) | add),
    shift_cost: (map(.cost) | add),
    shift_count: (map(.shifts) | add)
  })' > shift_summary.json

# Aggregate material transaction data by month
cat material_transactions.json | jq -r '
  group_by(.delivered_at[0:7]) | 
  map({
    month: .[0].delivered_at[0:7],
    material_tons: (map(.quantity) | add),
    load_count: length
  })' > material_summary.json

# Join and compare
jq -s '.[0] as $shifts | .[1] as $materials | 
  $shifts | map(. as $s | 
    ($materials | map(select(.month == $s.month)) | .[0]) as $m | 
    $s + {material_tons: ($m.material_tons // 0), load_count: ($m.load_count // 0)}
  ) | 
  map(. + {match_percent: ((.material_tons / .shift_tons) * 100 | round)})' \
  shift_summary.json material_summary.json
```

## Step 4: Interpret results

**If match is >95%:** Data is complete. The cost anomaly is real. Proceed to root cause analysis:
- Analyze billing patterns (hourly vs per-ton)
- Check material types (specialized vs standard)
- Review productivity metrics (tons/shift, trips/shift)
- Investigate customer/contract changes

**If match is <90%:** Investigate missing data:
- Check if material transactions are missing
- Verify if the trucker hauls for multiple customers
- Confirm customer filter is correct
- Look for data entry issues

## Example Output

A good validation report shows:

```
Month  Shift Tons  Material Tons  Match %  Shift Trips  Material Loads
-----  ----------  -------------  -------  -----------  --------------
01     1,245       1,238          99.4%    52           51
02     2,156       2,156          100.0%   89           89
03     1,834       1,821          99.3%    76           75
```

**Match >99%** = Data is complete, anomaly is real

## Key Insight

If shifts show 1,485 shifts but only 147 material loads were delivered (0.10 loads per shift vs normal 1.5-2.0 loads per shift), this indicates:
- Hourly billing with trucks staged on site (equipment rental)
- OR extremely inefficient operations
- NOT missing data

This validates that the cost anomaly requires investigation of billing structure, not data quality.
