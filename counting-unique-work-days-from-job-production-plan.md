---
title: Counting unique work days from job production plans
when: User asks how many days per year/month/period a customer had crews working
---

To count unique working days for a customer from job production plans:

1. **Always fetch ALL pages** - Production plans are paginated. Check the count and fetch multiple pages:
```bash
# Check first page
xbe view job-production-plans list --customer <ID> --start-on-min YYYY-MM-DD --start-on-max YYYY-MM-DD --json --limit 1000 | jq 'length'

# If you get 1000, there's more data - fetch next page with --offset 1000
# Continue until you get less than the limit
```

2. **Combine pages and count unique dates**:
```bash
(xbe view job-production-plans list --customer 89 --start-on-min 2025-01-01 --start-on-max 2025-12-31 --json --limit 1000 --offset 0 && \
 xbe view job-production-plans list --customer 89 --start-on-min 2025-01-01 --start-on-max 2025-12-31 --json --limit 1000 --offset 1000) | \
jq -s 'add | .[].start_on' | sort -u | wc -l
```

3. **Verify the date range** to ensure you have complete data:
```bash
# Check earliest and latest dates
... | jq -s 'add | .[].start_on' | sort -u | head -5
... | jq -s 'add | .[].start_on' | sort -u | tail -5
```

Key points:
- Don't assume the first page has all data
- Use `sort -u` to get unique dates
- The `jq -s 'add'` pattern combines multiple JSON arrays into one
- Verify your date range matches what the user asked for
