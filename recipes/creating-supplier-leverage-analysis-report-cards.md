---
title: Creating supplier leverage analysis report cards
when: When you need to analyze the power dynamics between a broker and their suppliers/truckers to inform supply chain management, pricing negotiations, or capital expenditure planning
---

# Creating Supplier Leverage Analysis Report Cards

This recipe shows how to create a comprehensive leverage analysis to determine negotiating power between a broker and their truckers/suppliers.

## Core Concept

Leverage analysis combines multiple dimensions:
1. **Volume/market share** - How much business does each trucker do?
2. **Lane diversity** - How many unique routes does each trucker operate?
3. **Concentration** - How dependent is the trucker on a few lanes vs. spread across many?
4. **Specialization** - What type of work (ready mix vs. paving vs. aggregate)?

## Step 1: Get Total Tonnage by Trucker

```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by trucker \
  --metrics tons_sum \
  --sort tons_sum:desc \
  --json
```

This gives you the volume each trucker handled.

## Step 2: Calculate Market Share

First get total tonnage across all operations:

```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --json | jq -r '.rows[] | "\(.tons_sum)"' | awk '{sum+=$1} END {printf "Total: %.2f\n", sum}'
```

Then calculate each trucker's percentage of total volume.

## Step 3: Analyze Lane Diversity per Trucker

For each major trucker, count their unique lanes:

```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter trucker=<trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by origin,destination \
  --metrics tons_sum,cycle_count \
  --sort tons_sum:desc \
  --json | jq '.rows | length'
```

This tells you how many different lanes (origin-destination pairs) they operate.

## Step 4: Calculate Concentration Ratio

Get the trucker's top lanes and calculate what percentage of their volume is in their top 1 or top 3 lanes:

```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter trucker=<trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by origin,destination \
  --metrics tons_sum \
  --sort tons_sum:desc \
  --json | jq -r '
    .rows | 
    (map(.tons_sum | tonumber) | add) as $total | 
    (.[0:3] | map(.tons_sum | tonumber) | add) as $top3 | 
    "Top 3 concentration: \(($top3 / $total * 100 | floor))%"'
```

High concentration (>60%) means the trucker is highly dependent on a few lanes with you.

## Step 5: Identify Specialization

Look at the lane names (origin/destination) to categorize:
- **Ready mix**: Quarry/mine to construction sites
- **Paving/asphalt**: Plant to road projects, overlays, DOT work
- **Aggregate**: Quarry to plant or stockpile

Paving work is seasonal and time-sensitive (higher trucker leverage).
Ready mix has more supplier alternatives (lower trucker leverage).

## Leverage Scoring Framework

**Trucker has leverage (low score 1-4) when:**
- High volume (>5% of external capacity)
- High lane diversity (>40 unique lanes)
- Low concentration (<30% in top 3 lanes)
- Specialized in time-sensitive work (paving)

**Broker has leverage (high score 7-10) when:**
- Low volume (<3% of external capacity)
- Low lane diversity (<15 unique lanes)
- High concentration (>60% in top 3 lanes)
- Focused on commodity work (ready mix)

**Balanced (score 5-6) when:**
- Moderate volume and diversity
- Medium concentration (30-60%)
- Mixed work types

## Strategic Interpretation

**For high-leverage truckers (they have power):**
- Consider capex to add internal fleet capacity in those lanes
- Negotiate long-term contracts with volume guarantees
- Develop backup/alternative haulers
- Accept premium pricing during peak seasons

**For low-leverage truckers (you have power):**
- Aggressive rate negotiations
- Annual rebid processes
- Use as flex capacity (first to cut in slow periods)
- Consolidate volume across fewer vendors

## Complete Analysis Output

Structure your report card with:
1. Executive summary (total volumes, internal vs. external split)
2. Tier 1: Critical suppliers (trucker has leverage)
3. Tier 2: Balanced relationships
4. Tier 3: Suppliers where broker dominates
5. Strategic recommendations (capex, supplier management, risk mitigation)

Each trucker entry should include:
- Volume and market share %
- Lane diversity count
- Concentration ratio
- Leverage score and reasoning
- Capex/strategic implications
