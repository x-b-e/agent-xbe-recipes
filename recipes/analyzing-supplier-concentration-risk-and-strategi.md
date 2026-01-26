---
title: Analyzing supplier concentration risk and strategic dependency
when: When you need to assess how dependent a broker is on a specific trucker (or vice versa), quantify concentration risk, evaluate alternative capacity, and develop risk mitigation strategies
---

# Analyzing supplier concentration risk and strategic dependency

## Overview
This recipe shows how to conduct a comprehensive supplier concentration analysis to understand dependency between a broker and a key trucker, quantify the risk, and develop mitigation strategies.

## Step 1: Identify the entities

Find the broker and trucker IDs:

```bash
# Find broker ID
xbe view brokers list --company-name "<broker-name>" --json

# Find trucker ID
xbe view truckers list --name "<trucker-name>" --json
```

## Step 2: Assess broker's dependency on the trucker

Get the broker's full trucker mix to see market share:

```bash
# Get all truckers for the broker, sorted by volume
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by trucker \
  --metrics tons_sum \
  --sort tons_sum:desc \
  --limit 50 \
  --json
```

**Note:** If you get a 503 timeout error with large date ranges (e.g., 2+ years), break it into smaller periods:
- Current year YTD
- Previous full year
- Recent quarter for trend analysis

Calculate market share:

```bash
# Get total broker volume
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --metrics tons_sum \
  --json | jq -r '.rows[0].tons_sum'

# Calculate percentage: (trucker tons / total tons) * 100
```

Key metrics to calculate:
- **Trucker's market share** with the broker (e.g., 51% = high concentration risk)
- **Gap to #2 trucker** (e.g., 5.8x larger = significant dependency)
- **Alternative capacity** (sum of next 5-10 truckers)

## Step 3: Assess trucker's dependency on the broker

Check if the trucker works with other brokers:

```bash
# Get all brokers the trucker works with
xbe summarize lane-summary create \
  --filter trucker=<trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by broker \
  --metrics tons_sum \
  --sort tons_sum:desc \
  --json
```

If the trucker shows 100% concentration with one broker, the dependency is mutual.

## Step 4: Analyze recent trends

Check if concentration is increasing or stable:

```bash
# Get recent quarter data
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<recent-quarter-start> \
  --filter date_max=<recent-quarter-end> \
  --group-by trucker \
  --metrics tons_sum \
  --sort tons_sum:desc \
  --limit 10 \
  --json

# Get monthly trend for the specific trucker
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter trucker=<trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by month \
  --metrics tons_sum \
  --json
```

## Step 5: Assess lane concentration

Understand if the trucker is concentrated on a few lanes or diversified:

```bash
# Get top lanes for the trucker with this broker
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter trucker=<trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by origin,destination \
  --metrics tons_sum \
  --sort tons_sum:desc \
  --limit 20 \
  --json
```

Calculate lane concentration:
- Top 1 lane as % of trucker's volume
- Top 3 lanes as % of trucker's volume

## Step 6: Evaluate alternative capacity

For each alternative trucker, assess their capacity to scale:

```bash
# Get each alternative trucker's volume with the broker
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter trucker=<alternative-trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --metrics tons_sum,trips_sum \
  --json
```

Key assessment:
- What % of the primary trucker's volume could each alternative handle?
- If all top 5 alternatives doubled capacity, what % of primary trucker could they replace?
- Are "unmanaged truckers" (customer-owned) inflating the alternative count?

## Step 7: Compile risk assessment

Create a summary with:

1. **Dependency metrics:**
   - Primary trucker's market share (High risk: >40%)
   - Mutual dependency (Does trucker show 100% concentration?)
   - Trend (Increasing, stable, or decreasing?)

2. **Alternative capacity:**
   - Aggregate capacity of top 5 alternatives
   - Number of truckers needed to replace primary
   - Backfill feasibility (Could alternatives scale quickly?)

3. **Lane-specific exposure:**
   - Top 3 lanes concentration
   - Geographic concentration
   - Material/equipment specialization

4. **Strategic impact assessment:**
   - Immediate capacity gap if relationship ends
   - Operational continuity risk
   - Customer impact (which customers rely on these lanes?)

## Step 8: Research and mitigation priorities

Based on risk level, develop a research and mitigation plan:

**Priority 1 - Relationship intelligence:**
- Contract terms and renewal dates
- Service quality and dispute history
- Financial stability of the trucker
- Strategic intentions (do they want to diversify?)

**Priority 2 - Market capacity:**
- Interview alternative truckers on scale capacity
- Geographic capacity mapping
- Lane-by-lane replacement feasibility

**Priority 3 - Strategic options:**
- Diversification targets (e.g., reduce from 51% to 35%)
- Contingency planning (30/60/90 day response plans)
- Relationship investment vs gradual reduction

**Priority 4 - Structural changes:**
- Owned/controlled capacity evaluation
- Technology improvements to attract diverse truckers
- M&A opportunities

## Risk level guidelines

- **High risk:** >40% concentration, mutual 100% dependency, limited alternatives
- **Moderate risk:** 25-40% concentration, some alternatives exist
- **Low risk:** <25% concentration, many viable alternatives

## Common patterns

- **Unmanaged truckers** in top 10 list may indicate capacity constraints (customers bringing their own trucks)
- **100% mutual dependency** suggests both parties benefit but broker lacks negotiation leverage
- **Stable multi-year concentration** suggests relationship is working but risk hasn't been addressed
