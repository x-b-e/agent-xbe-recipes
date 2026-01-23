---
title: Querying job production plans for future dates
when: When you need to find job production plans scheduled for tomorrow or other future dates, and need to handle the case where no plans exist yet
---

# Querying job production plans for future dates

When querying for job production plans scheduled for future dates (like tomorrow), be aware that plans may not exist yet if they haven't been created.

## Basic Future Date Query

```bash
xbe view job-production-plans list --start-on <future-date> --customer <customer-id>
```

## Handling Empty Results

If no plans are found for the future date:

1. **Check if plans exist for recent past dates** to verify the customer has active operations:
```bash
# Check yesterday
xbe view job-production-plans list --start-on <yesterday-date> --customer <customer-id>

# Check a date range
xbe view job-production-plans list --start-on-min <start-date> --start-on-max <end-date> --customer <customer-id>
```

2. **Explain to the user** that production plans may not be created yet for future dates, as they are typically created close to the actual work date.

3. **Offer alternatives**:
   - Show recent completed plans to demonstrate typical activity patterns
   - Suggest checking again closer to the target date
   - Search without date filters to see when the customer has active plans

## Date Calculations

When working with relative dates:
- Today: 2026-01-23 (use the current date from context)
- Tomorrow: Add 1 day
- Yesterday: Subtract 1 day

Use YYYY-MM-DD format for all date parameters.
