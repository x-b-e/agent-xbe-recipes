---
title: Analyze broker lanes and tonnage by trucker
when: User asks for lane analysis, tonnage summaries, volume reports by trucker, or shipping patterns for a broker
---

# Analyze Broker Lanes and Tonnage by Trucker

## Steps:

1. **Find the broker ID** by searching for the broker name:
```bash
xbe view brokers list --company-name "<broker-name>" --json
```

2. **Fetch material transactions** for the broker over a date range:
```bash
xbe view material-transactions list \
  --broker-id <broker-id> \
  --start-date YYYY-MM-DD \
  --end-date YYYY-MM-DD \
  --json
```

Note: Use appropriate date ranges (e.g., 1 year: 2025-01-20 to 2026-01-20)

3. **Analyze the data** using the analyze-lanes script:
```bash
node scripts/analyze-lanes.js <json-file>
```

This script aggregates transactions by trucker and lane, showing:
- Total tons per trucker
- Number of unique lanes
- Top 5 lanes by tonnage for each trucker
- Load counts

## Output Format:
The analysis shows truckers ranked by total tonnage, with their top lanes showing origin → destination with tons and load counts.

## Tips:
- Use `--json` flag to get machine-readable output for further processing
- The analyze-lanes script is located in the scripts/ directory
- Lanes are defined as origin → destination pairs
- Default unmanaged truckers appear separately in results
