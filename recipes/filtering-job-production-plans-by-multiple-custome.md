---
title: Filtering job production plans by multiple customers
when: When you need to find job production plans for multiple related customers (e.g., different entities within the same group like TrueRock Construction, TrueRock Materials, etc.)
---

# Filtering job production plans by multiple customers

## Overview

When filtering job production plans by customer, you can use comma-separated customer IDs. However, if no results are returned for a specific date, it doesn't necessarily mean there's an issue with your filter - there may simply be no plans for those customers on that date.

## Finding customer IDs

First, find the relevant customer IDs:

```bash
xbe view customers list --name <customer-name> --json
```

This often returns multiple related entities (e.g., "TrueRock Construction", "TrueRock Materials", "TrueRock Transport").

## Filtering by multiple customers

Use comma-separated customer IDs in the `--customer` flag:

```bash
xbe view job-production-plans list --start-on <date> --customer <id1>,<id2>,<id3>,<id4>
```

## Troubleshooting empty results

If you get no results for a specific date, verify the issue systematically:

1. **Check if ANY plans exist for the target date:**
   ```bash
   xbe view job-production-plans list --start-on <date> --limit 5
   ```

2. **If plans exist, check a broader date range for your customers:**
   ```bash
   xbe view job-production-plans list --start-on-min <start-date> --start-on-max <end-date> --customer <id1>,<id2>,<id3>
   ```

3. **If plans exist in the broader range but not on your target date:**
   - The customers simply have no plans scheduled for that specific date
   - This is normal and not a filter issue

## Alternative: Search by name

You can also search by job name or number using the `--q` flag:

```bash
xbe view job-production-plans list --start-on <date> --q "<search-term>"
```

Note: This searches job names and numbers, not customer names.

## Key insights

- Empty results doesn't mean your filter is wrong - it may mean no plans exist for those customers on that date
- Always verify with a broader date range to confirm the customers exist in the system
- Customer filtering works as expected; it's the date availability that varies
