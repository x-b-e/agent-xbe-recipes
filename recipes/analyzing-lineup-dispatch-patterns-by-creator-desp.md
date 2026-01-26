---
title: Analyzing lineup dispatch patterns by creator despite data model limitations
when: When you need to analyze who created lineup dispatches or understand dispatch patterns by creator, but the lineup-dispatch resource has null created-by-id fields. This is the complete workflow to work around the data model limitation.
---

## Problem

The `lineup-dispatches` resource often has `null` values in both `lineup-id` and `created-by-id` fields, making it impossible to directly filter dispatches by creator.

## Solution

Work around this by:
1. Finding lineups created by the target user(s)
2. Extracting lineup IDs
3. Filtering dispatches that belong to those lineups
4. Analyzing the results (counts, temporal patterns, etc.)

## Step-by-Step Workflow

### 1. Find lineups by creator and extract IDs

```bash
# Get lineup IDs for a specific creator
xbe view lineups list \
  --created-by-id <user-id> \
  --created-at-min <start-date> \
  --created-at-max <end-date> \
  --fields created-by \
  --json \
  --limit 1000 \
  | jq '[.[].id]' > /tmp/lineup_ids.json
```

### 2. Filter dispatches by lineup IDs and analyze

Once you have the lineup IDs, you can:

**Count total dispatches:**
```bash
xbe view lineup-dispatches list \
  --created-at-min <start-date> \
  --created-at-max <end-date> \
  --fields lineup,created-by \
  --json \
  --limit 1000 \
  | jq --slurpfile lineup_ids /tmp/lineup_ids.json '
    [.[] | select(.["lineup-id"] as $lid | $lineup_ids[0] | contains([$lid]))] | length'
```

**Group by date:**
```bash
xbe view lineup-dispatches list \
  --created-at-min <start-date> \
  --created-at-max <end-date> \
  --fields lineup,created-by,created-at \
  --json \
  --limit 1000 \
  | jq --slurpfile lineup_ids /tmp/lineup_ids.json '
    [.[] | select(.["lineup-id"] as $lid | $lineup_ids[0] | contains([$lid]))] 
    | group_by(.["created-at"][0:10]) 
    | map({date: .[0]["created-at"][0:10], count: length})'
```

**Group by day of week:**
```bash
# Iterate through dates and use date command to get day of week
for day in {01..31}; do 
  date_str="<year>-<month>-${day}"
  dow=$(date -j -f "%Y-%m-%d" "$date_str" "+%u:%A" 2>/dev/null || echo "")
  if [ -n "$dow" ]; then
    count=$(xbe view lineup-dispatches list \
      --created-at-min <year>-<month>-${day}T00:00:00Z \
      --created-at-max <year>-<month>-${day}T23:59:59Z \
      --fields lineup,created-by \
      --json \
      --limit 1000 2>&1 \
      | jq --slurpfile lineup_ids /tmp/lineup_ids.json '
        [.[] | select(.["lineup-id"] as $lid | $lineup_ids[0] | contains([$lid]))] | length')
    if [ "$count" -gt 0 ]; then
      echo "$dow|$count"
    fi
  fi
done | awk -F'|' '{day=$1; split(day,a,":"); dow[a[2]]+=$2; order[a[1]]=a[2]} 
  END {for(i=1;i<=7;i++) if(order[i]) printf "%s: %d\n", order[i], dow[order[i]]}'
```

**Group by hour of day:**
```bash
# Iterate through each hour and sum across all days
for hour in {00..23}; do
  count=0
  for day in {01..31}; do
    day_count=$(xbe view lineup-dispatches list \
      --created-at-min <year>-<month>-${day}T${hour}:00:00Z \
      --created-at-max <year>-<month>-${day}T${hour}:59:59Z \
      --fields lineup,created-by \
      --json \
      --limit 1000 2>&1 \
      | jq --slurpfile lineup_ids /tmp/lineup_ids.json '
        [.[] | select(.["lineup-id"] as $lid | $lineup_ids[0] | contains([$lid]))] | length' \
      2>/dev/null || echo 0)
    count=$((count + day_count))
  done
  if [ "$count" -gt 0 ]; then
    echo "Hour ${hour}:00-${hour}:59: $count dispatches"
  fi
done
```

### 3. For multiple creators comparison

If comparing multiple creators:

```bash
# Get lineups for each creator separately
for user_id in <user-id-1> <user-id-2> <user-id-3>; do
  xbe view lineups list \
    --created-by-id $user_id \
    --created-at-min <start-date> \
    --created-at-max <end-date> \
    --fields created-by \
    --json \
    --limit 1000 \
    | jq -r --arg uid "$user_id" \
      '.[] | "\($uid),\(.id)"' >> /tmp/lineup_user_mapping.csv
done

# Then aggregate dispatches per user
# (implementation depends on specific analysis needs)
```

## Key Points

- Always save lineup IDs to a temp file for reuse across multiple queries
- Use `jq --slurpfile` to load the lineup IDs array and filter with `contains([$lid])`
- For temporal analysis, you'll need to iterate through time periods and query each separately
- The dispatch queries can return large result sets, so use `--limit 1000` and consider pagination if needed
- Times are in UTC, so adjust for local timezone if needed for reporting

## Related Recipes

- understanding-lineup-dispatch-data-model-limitations.md - Explains why this workaround is necessary
- paginating-through-large-result-sets.md - If you have more than 1000 lineups or dispatches
