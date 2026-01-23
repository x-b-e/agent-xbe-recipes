---
title: Analyzing seasonal patterns in concrete mix sales
when: When you need to identify seasonal patterns in concrete/material sales, including which mix types are preferred in different seasons (winter vs summer), volume distribution across months, and material-specific seasonal behaviors
---

# Analyzing seasonal patterns in concrete mix sales

This recipe shows how to analyze seasonal patterns in concrete sales data, identifying both volume seasonality and mix-type preferences by season.

## Basic approach

1. **Get material transaction data grouped by month and material type**

```bash
xbe summarize material-transaction-summary create \
  --broker-id <broker-id> \
  --date-range-start <start-date> \
  --date-range-end <end-date> \
  --group-by month \
  --group-by material_type_id \
  --json > /tmp/monthly_mixes.json
```

2. **Calculate monthly volume totals**

```bash
cat /tmp/monthly_mixes.json | jq '.values | 
  map(select(.[1] and (.[1] | type == "string") and (.[1] | startswith("Concrete:")))) | 
  group_by(.[0]) | 
  map({month: (.[0][0] | tonumber), cubic_yards: ((map(.[3]) | add) / 2)}) | 
  sort_by(.month)'
```

3. **Identify winter vs summer mix preferences**

```bash
cat /tmp/monthly_mixes.json | jq '.values | 
  map(select(.[1] and (.[1] | type == "string") and (.[1] | startswith("Concrete:")))) | 
  map({month: (.[0] | tonumber), mix: .[1], tons: .[3]}) | 
  group_by(.mix) | 
  map({
    mix: .[0].mix, 
    winter_tons: (map(select(.month == 1 or .month == 2 or .month == 12)) | map(.tons) | add // 0), 
    summer_tons: (map(select(.month >= 6 and .month <= 8)) | map(.tons) | add // 0), 
    total_tons: (map(.tons) | add)
  }) | 
  map(select(.total_tons > <minimum-tons>)) | 
  sort_by(-((.winter_tons / .total_tons) - (.summer_tons / .total_tons))) | 
  map({
    mix: (.mix | sub("^Concrete:"; "")), 
    winter_pct: ((.winter_tons / .total_tons * 100) | round), 
    summer_pct: ((.summer_tons / .total_tons * 100) | round), 
    total_cy: ((.total_tons / 2) | round)
  }) | 
  {most_winter: .[0:10], most_summer: .[-10:]}'
```

4. **Create a visual monthly volume chart**

```bash
cat /tmp/monthly_mixes.json | jq '.values | 
  map(select(.[1] and (.[1] | type == "string") and (.[1] | startswith("Concrete:")))) | 
  group_by(.[0]) | 
  map({month: (.[0][0] | tonumber), cubic_yards: ((map(.[3]) | add) / 2)}) | 
  sort_by(.month) | 
  map("\(.month | ["", "Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"][.]) \(.cubic_yards | round | tostring | . + " cy") \(("#" * (.cubic_yards / <scale-factor> | floor)) // "")")' -r
```

5. **Analyze specific material characteristics by season**

For example, to compare TRI BLEND vs non-TRI BLEND mixes:

```bash
cat /tmp/monthly_mixes.json | jq '.values | 
  map(select(.[1] and (.[1] | type == "string") and (.[1] | startswith("Concrete:")))) | 
  map({month: (.[0] | tonumber), mix: .[1], tons: .[3]}) | 
  group_by(.mix) | 
  map({
    mix: .[0].mix, 
    has_tri_blend: (.[0].mix | test("TRI BLEND"; "i")), 
    winter_tons: (map(select(.month == 1 or .month == 2 or .month == 12)) | map(.tons) | add // 0), 
    summer_tons: (map(select(.month >= 6 and .month <= 8)) | map(.tons) | add // 0), 
    total_tons: (map(.tons) | add)
  }) | 
  group_by(.has_tri_blend) | 
  map({
    type: (if .[0].has_tri_blend then "<material-type> mixes" else "Other" end), 
    winter_tons: (map(.winter_tons) | add), 
    summer_tons: (map(.summer_tons) | add), 
    total_tons: (map(.total_tons) | add), 
    winter_cy: ((map(.winter_tons) | add) / 2 | round), 
    summer_cy: ((map(.summer_tons) | add) / 2 | round)
  })'
```

## Key insights this analysis reveals

- **Volume seasonality**: Monthly volume distribution shows construction season peaks and winter slowdowns
- **Mix preferences by season**: Certain mixes are heavily winter-focused (cold weather formulations) while others are summer-only (warm weather products)
- **Material characteristics**: Specific ingredients or admixtures (TRI BLEND, SLAG, high early strength) show strong seasonal patterns
- **Climate-driven behavior**: The degree of seasonality indicates the impact of freeze-thaw cycles on construction activity

## Tips

- Define winter as months 1, 2, 12 and summer as months 6, 7, 8 for clearest seasonal contrast
- Filter to mixes with significant total volume (e.g., >5000 tons) to avoid noise from rarely-used formulations
- Look for mixes with >80% seasonal concentration - these are likely purpose-built for specific weather conditions
- Compare percentages rather than absolute volumes to identify true seasonal preferences vs overall popularity
- Use `test("PATTERN"; "i")` for case-insensitive material characteristic matching (SLAG, TRI BLEND, etc.)
