---
title: Comparing trucker spend and efficiency between customers
when: When you need to compare trucking costs, efficiency, and supplier mix between two or more customers/brokers to identify cost optimization opportunities
---

# Comparing trucker spend and efficiency between customers

When comparing trucking operations between customers/brokers, you need to combine multiple data sources to get a complete picture.

## Data sources needed

1. **Material transactions** - for actual tonnage moved and load counts
2. **Trucker spend summaries** - for costs and shift counts
3. **Customer/broker IDs** - to filter correctly

## Step-by-step approach

### 1. Identify all related customer entities

Many companies have multiple customer records (e.g., "TrueRock Construction", "TrueRock Materials", "TrueRock Transport"). Get all IDs:

```bash
xbe view customers list --name <company-name> --json
```

Extract all IDs to use in comma-separated filters.

### 2. Get trucker spend data

For each customer group, get trucker-level spend aggregates:

```bash
xbe view trucker-spend-summaries list \
  --start-on-min <start-date> \
  --start-on-max <end-date> \
  --customer <customer-id-1>,<customer-id-2>,<customer-id-3> \
  --group-by trucker_id \
  --json > <customer-name>_trucker_spend.json
```

### 3. Get material transaction details

To understand what materials are being hauled and by whom:

```bash
xbe view material-transactions list \
  --datetime-min <start-datetime> \
  --datetime-max <end-datetime> \
  --customer <customer-id> \
  --material-category <material-category> \
  --json > <customer-name>_material_transactions.json
```

### 4. Calculate key metrics

From the combined data, calculate:

- **Cost per ton**: Total spend ÷ Total tonnage
- **Tons per shift**: Total tonnage ÷ Total shifts
- **Average load size**: Total tonnage ÷ Number of loads (from material transactions)
- **Fleet concentration**: Top trucker's percentage of total volume
- **Unmanaged exposure**: Shifts/tonnage with no cost tracking

### 5. Identify optimization opportunities

Look for:

- **Productivity gaps**: Significant differences in tons/shift between customers
- **Cost outliers**: Truckers with unusually high $/ton rates
- **Underutilized efficient truckers**: Low-cost truckers with low volumes
- **Cost tracking gaps**: High volumes from "unmanaged" truckers

## Example analysis patterns

### Pattern 1: Internal fleet comparison

Compare the primary internal trucking entity between customers:

```bash
# Get TrueRock Logistics performance
jq '.[] | select(.trucker_name | contains("TrueRock Logistics")) | 
   {shifts, tons, spend, cost_per_ton: (.spend / .tons)}' \
   truerock_trucker_spend.json

# Get Scot Heller Trucking performance  
jq '.[] | select(.trucker_name | contains("Scot Heller")) | 
   {shifts, tons, spend, cost_per_ton: (.spend / .tons)}' \
   superior_bowen_trucker_spend.json
```

### Pattern 2: Load size analysis

From material transactions, calculate average loads by trucker:

```bash
jq 'group_by(.trucker_name) | map({
  trucker: .[0].trucker_name,
  loads: length,
  tons: (map(.quantity) | add),
  avg_load: ((map(.quantity) | add) / length)
}) | sort_by(-.tons)' material_transactions.json
```

### Pattern 3: Cost variance identification

Find truckers with costs significantly above/below average:

```bash
jq 'map({trucker: .trucker_name, cost_per_ton: (.spend / .tons)}) | 
   . as $all | 
   ($all | map(.cost_per_ton) | add / length) as $avg |
   map(. + {variance: ((.cost_per_ton - $avg) / $avg * 100)}) |
   sort_by(-.variance)' trucker_spend.json
```

## Common findings to report

1. **Structural cost advantages**: When one customer has a dominant low-cost trucker relationship
2. **Productivity gaps**: Differences in tons/shift indicating operational efficiency issues
3. **Cost outliers**: Specific truckers requiring negotiation or replacement
4. **Optimization opportunities**: Quantified savings from matching best-in-class performance
5. **Data quality issues**: Significant unmanaged/untracked volumes

## Tips

- Always exclude unmanaged truckers from averages, but report them separately
- Calculate both total averages and managed-only averages
- For material-specific analysis (e.g., asphalt only), filter transactions by material category
- When comparing across customers, ensure date ranges are identical
- Watch for truckers doing specialized work (excavation, equipment movement) vs pure hauling
