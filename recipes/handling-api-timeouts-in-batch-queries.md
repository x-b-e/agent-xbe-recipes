---
title: Handling API timeouts in batch queries
when: When you're querying the XBE API in a loop (e.g., iterating through months) and some queries time out with 503 errors, causing JSON parsing errors in downstream processing
---

# Handling API timeouts in batch queries

## Problem
When running XBE API queries in a bash loop (e.g., querying each month separately), some queries may time out with `503 Service Unavailable` errors. If your script doesn't handle errors properly, the invalid JSON response gets passed to `jq`, causing parse errors and breaking your calculations.

## Symptoms
```bash
# Your script shows errors like:
Parse error: bad expression
Parse error: bad token

# And you see 503 errors in the output:
{"errors":[{"title":"SERVICE UNAVAILABLE","detail":"Request took too long..."}]}
request failed: 503 Service Unavailable
```

## Solution: Add error handling and retry logic

### Option 1: Check for errors before parsing
```bash
for month in {1..12}; do
  result=$(xbe summarize material-transaction-summary create \
    --filter broker=<broker-id> \
    --filter year=<year> \
    --filter month=$month \
    --json)
  
  # Check if the result contains an error
  if echo "$result" | grep -q '"errors"'; then
    echo "Month $month: Query failed, retrying..."
    sleep 2
    result=$(xbe summarize material-transaction-summary create \
      --filter broker=<broker-id> \
      --filter year=<year> \
      --filter month=$month \
      --json)
  fi
  
  # Only parse if we have valid JSON
  if echo "$result" | jq -e . >/dev/null 2>&1; then
    tons=$(echo "$result" | jq -r '.rows[0].tons_sum // 0')
    echo "Month $month: $tons tons"
  else
    echo "Month $month: Failed to get data"
  fi
done
```

### Option 2: Query each month individually when troubleshooting
Instead of running a loop, query failing months individually:

```bash
# Query the specific month that timed out
xbe summarize material-transaction-summary create \
  --group-by "" \
  --filter broker=<broker-id> \
  --filter year=<year> \
  --filter month=<month> \
  --filter "<your-filters>" \
  --json
```

Retrying individual queries often succeeds because server load varies.

### Option 3: Use `jq` default values
Always use `// 0` or similar default values in jq to handle missing data:

```bash
# This will return 0 if tons_sum is missing or null
tons=$(echo "$result" | jq -r '.rows[0].tons_sum // 0')

# For more robust handling:
tons=$(echo "$result" | jq -r '.rows[0].tons_sum // "N/A"')
```

## When timeouts are persistent
If specific months consistently time out:
1. They likely have much more data than other months (peak season)
2. Consider adding more specific filters to reduce the dataset
3. Break the month into smaller date ranges
4. Query during off-peak hours if possible

## Prevention
- Add `sleep 1` between queries in loops to avoid overwhelming the API
- Use `--limit` when appropriate to reduce query size
- Consider caching results for queries you'll need multiple times
