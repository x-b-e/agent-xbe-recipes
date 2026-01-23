---
title: Analyzing concrete sales by mix type
when: When you need to analyze concrete/ready-mix sales broken down by mix type, including handling the conversion from tons to cubic yards and dealing with large numbers of mix types
---

## Context
Concrete material transactions are stored in the XBE system with:
- Material names formatted as `Concrete:<code> (<description>)`
- Quantities tracked in tons (not cubic yards)
- Industry standard conversion: 1 cubic yard ≈ 2 tons
- Typically thousands of different mix types per broker

## Step-by-step workflow

### 1. Create material transaction summary for the broker
```bash
xbe summarize material-transaction-summary create \
  --broker-id "<broker-id>" \
  --start-date "<start-date>" \
  --end-date "<end-date>" \
  --group-by "material_type_id" \
  --json > /tmp/materials.json
```

### 2. Filter to concrete materials only
The material names in the summary start with "Concrete:" prefix:

```bash
cat /tmp/materials.json | jq -r '.values[] | select((.[0] // "") | type == "string" and startswith("Concrete:"))' > /tmp/concrete_only.json
```

**Key points:**
- Use `.[0] // ""` to handle null values safely
- Check `type == "string"` before using `startswith()` to avoid type errors
- The values array format is: `[material_name, transaction_count, tons]`

### 3. Aggregate by mix type and convert to cubic yards
```bash
cat /tmp/concrete_only.json | jq -s 'group_by(.[0]) | map({
  mix_type: .[0][0] | sub("^Concrete:"; ""),
  transaction_count: (map(.[1]) | add),
  tons: (map(.[2]) | add),
  cubic_yards: ((map(.[2]) | add) / 2)
}) | sort_by(-.cubic_yards)' > /tmp/concrete_breakdown.json
```

**Key points:**
- Use `jq -s` to slurp all records into an array for grouping
- `group_by(.[0])` groups by material name
- Remove "Concrete:" prefix with `sub("^Concrete:"; "")`
- Convert tons to cubic yards by dividing by 2
- Sort by cubic yards descending with `sort_by(-.cubic_yards)`

### 4. Generate summary statistics
```bash
cat /tmp/concrete_breakdown.json | jq '{
  total_cubic_yards: (map(.cubic_yards) | add),
  total_tons: (map(.tons) | add),
  total_transactions: (map(.transaction_count) | add),
  mix_type_count: length,
  top_20: .[0:20]
}'
```

## Output interpretation
- **cubic_yards**: Industry standard unit for concrete volume
- **tons**: Raw weight from XBE system
- **transaction_count**: Number of delivery tickets/loads
- **mix_type**: The concrete mix design code and description

## Common patterns in mix type names
- Strength ratings: `4K`, `5K`, `6K` (4000 psi, 5000 psi, etc.)
- Slump ranges: `(3-5" SLUMP)`, `(1-3" SLUMP)`
- Aggregate types: `SLAG`, `TRI BLEND`, aggregate size like `3/4`
- Special designations: `CO2` (reduced carbon), `MOD` (modified), `WALL` (wall mix)
- Codes often indicate: strength-aggregate-slump-additives

## Tips
- With 1000+ mix types, focus on top performers (top 20-50 typically represent 30-50% of volume)
- Group similar mixes for higher-level analysis if needed
- The 2 tons/cy conversion is an approximation; actual density varies by mix design but is acceptable for business analysis
