---
title: Comparing operational performance between customer groups
when: When you need to compare operational performance metrics (achievement rates, timing, costs, trends) between two customer groups or brokers across multiple dimensions and time periods
---

# Comparing operational performance between customer groups

This recipe shows how to perform comprehensive operational performance comparisons between customer groups (or brokers) using job production plan data.

## Key steps

### 1. Identify all related customer entities

Customer groups often have multiple legal entities. Find all of them:

```bash
xbe view customers list --name <customer-group-name> --json
```

Extract the IDs for use in filters (e.g., `99,102,8463,12812`).

### 2. Query job production plans for each customer group

Use comma-separated customer IDs to get all plans for a group:

```bash
xbe view job-production-plans list \
  --start-on-min <start-date> \
  --start-on-max <end-date> \
  --customer <customer-id-1>,<customer-id-2>,<customer-id-3> \
  --json
```

**Important**: For large date ranges (3+ months), be prepared for timeouts. Consider using `--limit` or breaking into smaller date ranges.

### 3. Compute key performance metrics

For each customer group, calculate:

**Volume metrics:**
- Total plans by status (approved, complete, cancelled, abandoned)
- Goal tonnage vs actual delivered tonnage
- Achievement rate (actual/goal %)

**Timing metrics:**
- Average start time offset (planned vs actual start)
- On-time percentage

**Cost metrics:**
- Cost per ton (from truck spend / tonnage)
- Total truck hours and spend

**Quality metrics:**
- Surplus percentage (actual vs planned surplus)
- Production incident duration

**Workflow metrics:**
- Status distribution (editing, submitted, approved, complete, cancelled)
- Cancellation/abandonment rate

### 4. Break down by time dimensions

Analyze trends across:
- **Overall period**: Aggregate metrics for entire date range
- **Monthly**: Break down by month to identify trends
- **Weekly**: Recent weeks to see current trajectory

### 5. Break down by entity

If a customer group has multiple entities, analyze each separately to identify which entities perform well vs poorly.

### 6. Create comparison report

Organize findings into sections:
- Executive summary (high-level comparison)
- Head-to-head metrics table
- Key differences analysis
- Trend analysis (monthly/weekly)
- Entity-level breakdown
- Critical insights
- Recommendations

## Example workflow

```bash
# Find Customer Group A entities
xbe view customers list --name "<customer-group-a>" --json > customer_a.json

# Find Customer Group B entities  
xbe view customers list --name "<customer-group-b>" --json > customer_b.json

# Get plans for Customer Group A (using extracted IDs)
xbe view job-production-plans list \
  --start-on-min <start-date> \
  --start-on-max <end-date> \
  --customer <id-1>,<id-2>,<id-3> \
  --json > plans_a.json

# Get plans for Customer Group B
xbe view job-production-plans list \
  --start-on-min <start-date> \
  --start-on-max <end-date> \
  --customer <id-4>,<id-5> \
  --json > plans_b.json

# Process with jq to compute metrics
jq '[.[] | select(.status == "approved")] | 
  {total: length, 
   goal_tons: ([.[].goal_quantity_in_tons] | add),
   actual_tons: ([.[].actual_quantity_in_tons] | add),
   achievement: (([.[].actual_quantity_in_tons] | add) / ([.[].goal_quantity_in_tons] | add) * 100)}' plans_a.json
```

## Important considerations

**Status workflow differences**: Different customers may use status workflows differently (e.g., some mark plans as "complete", others keep them "approved" indefinitely). Analyze both approved and complete plans separately.

**Equipment movement plans**: Some customers have "equipment movement" plans with 0 tonnage that skew cancellation rates. Consider filtering these out with `--is-only-for-equipment-movement false`.

**Date range handling**: For 3+ month ranges, consider breaking into monthly chunks and aggregating results to avoid timeouts.

**Entity-level insights**: Always break down by individual customer entities within a group - performance can vary dramatically between entities (e.g., "Construction" vs "Materials" divisions).
