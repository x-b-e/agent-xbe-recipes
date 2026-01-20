---
title: Use jq file for complex JSON analysis
when: you need to run complex jq queries with multiple filters, grouping, or transformations, especially when encountering quoting or escaping issues
---

# Use jq File for Complex JSON Analysis

## When to use
Use when you need to run complex jq queries with multiple filters, grouping, or transformations, especially when encountering quoting or escaping issues in bash.

## Problem
Complex jq queries with multiple conditions, string comparisons, and operators often fail in bash due to:
- Shell quoting conflicts
- Special character escaping issues  
- Readability and maintainability problems

## Solution
Write the jq program to a file and execute it with `-f`:

```bash
# 1. Write jq program to a file
cat > /tmp/analysis.jq <<'EOF'
map(select(
  .quantity != "" and
  .quantity != null
))
| map({
    field1: .field1,
    field2: .field2,
    numeric: (.string_field | tonumber)
  })
| group_by(.field1)
| map({
    key: .[0].field1,
    total: (map(.numeric) | add),
    count: length
  })
| sort_by(-.total)
EOF

# 2. Execute with -f flag
jq -r -f /tmp/analysis.jq input.json
```

## Benefits
- No quoting or escaping issues
- Easier to read and debug
- Can include comments in the jq file
- Reusable for multiple runs

## Formatting output
Combine with `awk` for formatted tables:
```bash
jq -r -f /tmp/analysis.jq input.json | \
  awk 'BEGIN {FS="\t"; printf "%-30s %10s\n", "FIELD", "VALUE"} \
       {printf "%-30s %10.2f\n", $1, $2}'
```
