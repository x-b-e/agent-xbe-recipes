---
title: Investigating trucking cost anomalies by analyzing material composition
when: When you need to determine if a trucker's high cost per ton is justified by hauling different materials, specialty loads, or unusual work compared to other truckers
---

# Investigating trucking cost anomalies by analyzing material composition

When a trucker has significantly higher costs per ton than others, you need to determine if they're doing fundamentally different work that justifies the premium.

## Step 1: Get baseline cost metrics

First, establish the cost anomaly:

```bash
xbe view lane-summaries \
  --start-on-min <start-date> \
  --start-on-max <end-date> \
  --broker <broker-id> \
  --group-by trucker \
  --include-effective-cost-per-hour \
  --json
```

This shows cost per ton, tons per hour, and effective hourly rate by trucker.

## Step 2: Analyze material composition for the expensive trucker

Pull all material transactions to see what they're actually hauling:

```bash
xbe view material-transactions list \
  --created-at-min <start-date> \
  --created-at-max <end-date> \
  --trucker <trucker-id> \
  --limit 9999 \
  --json > expensive_trucker_materials.json
```

## Step 3: Summarize material types and load characteristics

Use jq to group by material type and calculate load statistics:

```bash
cat expensive_trucker_materials.json | jq -r '
  group_by(.material_type.name) | 
  map({
    material: .[0].material_type.name,
    loads: length,
    total_tons: (map(.quantity_tons) | add),
    avg_tons_per_load: ((map(.quantity_tons) | add) / length)
  }) |
  sort_by(-.total_tons) |
  ["MATERIAL", "LOADS", "TONS", "AVG_TONS/LOAD"],
  ["---", "---", "---", "---"],
  (.[] | [.material, .loads, (.total_tons | floor), (.avg_tons_per_load | floor)]) |
  @tsv
' | column -t
```

## Step 4: Compare to baseline truckers

Repeat steps 2-3 for comparison truckers (e.g., the average-cost trucker):

```bash
xbe view material-transactions list \
  --created-at-min <start-date> \
  --created-at-max <end-date> \
  --trucker <comparison-trucker-id> \
  --limit 9999 \
  --json > comparison_trucker_materials.json
```

Then run the same jq summary.

## Step 5: Look for material category patterns

If dealing with many material variants, group by category:

```bash
cat expensive_trucker_materials.json | jq -r '
  group_by(.material_type.category) | 
  map({
    category: .[0].material_type.category,
    loads: length,
    total_tons: (map(.quantity_tons) | add),
    percent: 0
  }) |
  . as $data |
  ($data | map(.total_tons) | add) as $total |
  $data | map(.percent = (.total_tons / $total * 100)) |
  sort_by(-.total_tons) |
  ["CATEGORY", "LOADS", "TONS", "% OF TOTAL"],
  (.[] | [.category, .loads, (.total_tons | floor), (.percent | floor)])
' | column -t
```

## Step 6: Compare load sizes

If materials are similar, check if load sizes differ (which might indicate specialty equipment):

```bash
# Compare average tons per load across truckers
cat expensive_trucker_materials.json | jq '
  {loads: length, total_tons: (map(.quantity_tons) | add)} |
  {avg_tons_per_load: (.total_tons / .loads)}
'

cat comparison_trucker_materials.json | jq '
  {loads: length, total_tons: (map(.quantity_tons) | add)} |
  {avg_tons_per_load: (.total_tons / .loads)}
'
```

## Key Decision Points

**Cost premium IS justified if:**
- Different material types (e.g., hazmat vs standard aggregate)
- Significantly different load sizes (e.g., 10 ton specialty vs 20 ton standard)
- Equipment-specific materials (e.g., equipment transport vs bulk materials)

**Cost premium is NOT justified if:**
- Same material types and categories
- Similar load sizes (within 10-20%)
- Same material categories (e.g., all asphalt, all aggregate)

In this case, the cost difference is a **productivity/efficiency problem**, not a work-type difference.

## Example Output Interpretation

If both truckers show:
- 70-80% Asphalt mixtures
- 20-30% Aggregates
- 18-24 tons per load average

Then they're doing the **same work**, and cost differences are due to:
- Fewer trips per shift
- Longer cycle times
- Inefficient routing
- Billing model issues

This justifies a productivity investigation rather than accepting the cost premium.
