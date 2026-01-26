---
title: Analyzing supplier concentration risk and strategic dependency with lane-level alternatives
when: When you need to assess how dependent a broker is on a specific trucker (or vice versa), quantify concentration risk, evaluate alternative capacity at both aggregate and lane levels, analyze origin-level dependencies, estimate fleet requirements for replacement scenarios, and develop risk mitigation strategies
---

# Analyzing supplier concentration risk and strategic dependency with lane-level alternatives

When analyzing concentration risk between a broker and trucker, you need multiple perspectives: overall volume dependency, lane-level alternatives, origin-level exclusivity, and replacement feasibility.

## Step 1: Identify the entities

Find the broker and trucker IDs:

```bash
xbe view brokers list --company-name "<broker-name>" --json
xbe view truckers list --name "<trucker-name>" --json
```

## Step 2: Calculate overall concentration (broker's dependency)

Get the broker's total tonnage broken down by all truckers to see concentration:

```bash
# For recent period (2025-2026)
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

**Key insight**: Use `--limit 50` to avoid timeouts on large datasets.

Calculate concentration ratio:
- Top trucker tonnage / Total broker tonnage = concentration %
- If >50%, this represents high strategic risk

## Step 3: Analyze the trucker's dependency on this broker

Get the trucker's total tonnage broken down by broker:

```bash
xbe summarize lane-summary create \
  --filter trucker=<trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by broker \
  --metrics tons_sum \
  --sort tons_sum:desc \
  --json
```

Calculate reverse dependency:
- This broker's tonnage / Trucker's total tonnage = reverse concentration %
- If high on both sides, this is a mutual dependency situation

## Step 4: Identify top lanes for detailed analysis

Get the trucker's top lanes for this broker to understand where volume is concentrated:

```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter trucker=<trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by origin_name,destination_name \
  --metrics tons_sum,trips_sum \
  --sort tons_sum:desc \
  --limit 20 \
  --json
```

Analyze lane concentration:
- Calculate what % of trucker's volume comes from top 3, top 5, top 10 lanes
- High lane concentration indicates vulnerability to specific route disruptions

## Step 5: Analyze alternative trucker capacity by destination

For each major destination plant, see what alternative truckers are already serving it:

```bash
# For a specific destination
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter destination_name="<destination-name>" \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by trucker \
  --metrics tons_sum \
  --sort tons_sum:desc \
  --json
```

**Critical analysis**: Calculate replacement capacity:
- Primary trucker tonnage to this destination
- Sum of all alternative trucker tonnage to this destination
- Replacement coverage % = alternatives / primary
- If <20%, this is HIGH RISK
- If 20-50%, MEDIUM RISK (would need 2-5x growth)
- If >50%, LOWER RISK

## Step 6: Analyze origin-level dependencies (critical!)

For each major origin the trucker uses, check if they're exclusive:

```bash
# Check all truckers serving a specific origin
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter origin_name="<origin-name>" \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by trucker \
  --metrics tons_sum \
  --sort tons_sum:desc \
  --json
```

**Critical finding**: If a trucker handles 95-100% of volume from a specific origin:
- This represents origin-level exclusivity
- Alternatives cannot easily serve these lanes without establishing presence at the origin
- This is often the highest-risk scenario

## Step 7: Assess alternative trucker total capacity

Get the total company size of each alternative trucker:

```bash
xbe summarize lane-summary create \
  --filter trucker=<alternative-trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --metrics tons_sum \
  --json
```

**Feasibility assessment**:
- Sum up alternative truckers' current tonnage on the broker's plants
- Compare to their TOTAL company tonnage
- Calculate required growth: (Primary trucker tonnage) / (Alternative current tonnage)
- If alternatives would need to grow by >5x their TOTAL company size, replacement is infeasible in short/medium term

## Step 8: Estimate fleet size requirements

Calculate the trucker's estimated fleet size:

```bash
# Get monthly driver counts
xbe summarize driver-summary create \
  --filter trucker=<trucker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by month \
  --metrics active_drivers_avg \
  --json
```

Rule of thumb:
- Average ~3 drivers per truck
- Peak season drivers / 3 = approximate fleet size
- This helps quantify the capital/operational scale needed for replacement

## Step 9: Synthesize risk assessment

Create a comprehensive risk report covering:

1. **Overall Concentration**: X% of broker volume, Y% of trucker volume
2. **Lane Concentration**: Top 10 lanes = Z% of volume
3. **Plant-Level Risk**: For each major plant:
   - Primary trucker tonnage and % of plant total
   - Alternative truckers' tonnage and % coverage
   - Replacement feasibility (LOW/MEDIUM/HIGH RISK)
4. **Origin-Level Risk**: Origins where trucker is exclusive (95-100%)
5. **Alternative Capacity Analysis**:
   - Total alternative tonnage on broker's plants
   - % of primary trucker this represents
   - Required growth factor (e.g., "6x their total company size")
   - Feasibility assessment
6. **Fleet Requirements**: Estimated trucks and drivers needed for replacement

## Step 10: Develop mitigation strategies

Based on risk level, prioritize:

**Short-term (if HIGH RISK):**
- [ ] Strengthen relationship/contract with primary trucker
- [ ] Identify specific lanes where alternatives could grow (start with highest coverage %)
- [ ] Establish alternative presence at exclusive origins
- [ ] Negotiate pricing/terms reflecting mutual dependency

**Medium-term:**
- [ ] Recruit new trucking partners from outside market
- [ ] Develop alternative origins for key materials
- [ ] Grow existing alternatives on specific lanes (target 2-3x growth on best candidates)
- [ ] Create contingency plans for partial capacity loss

**Long-term:**
- [ ] Consider vertical integration (acquire trucking capacity)
- [ ] Diversify origin-destination network
- [ ] Build strategic reserve capacity
- [ ] Develop multi-year alternative capacity growth plan

## Key Insights

- **Volume concentration alone is insufficient** - you must analyze lane-level alternatives and origin exclusivity
- **Origin-level exclusivity is often the highest risk** - alternatives can't easily serve lanes they don't already touch
- **Replacement feasibility depends on alternative truckers' total company size** - requiring 6-8x total company growth is unrealistic
- **Plant-level analysis reveals where alternatives exist** - start replacement efforts where you already have 20%+ coverage
- **Don't assume alternatives can simply "grow"** - validate their total capacity and growth constraints
