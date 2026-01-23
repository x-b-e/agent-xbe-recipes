---
title: Searching job production plans by customer name vs ID
when: When you need to find job production plans for a customer and filtering by customer ID returns no results, or when you want to search broadly vs filter precisely
---

# Searching job production plans by customer name vs ID

## Problem
Filtering job production plans by customer ID may not return the expected results, even when you know plans exist for that customer.

## Solution
Use the `--q` flag for broader search capabilities instead of the `--customer` ID filter.

### Search by name (broader)
```bash
xbe view job-production-plans list --start-on <date> --q "<customer-name>"
```

The `--q` flag searches across multiple fields including job names and descriptions, which may include customer references.

### Filter by customer ID (precise)
```bash
xbe view job-production-plans list --start-on <date> --customer <customer-id>
```

The `--customer` flag filters strictly by the customer ID associated with the plan.

## Key insight
These two approaches can return different results:
- `--q` searches text fields and may find plans where the customer name appears in job names or descriptions
- `--customer` filters by the explicit customer relationship ID

If you're looking for plans related to a customer organization but `--customer` returns no results, try `--q` with the customer name instead.

## Example workflow
```bash
# First, find customer IDs
xbe view customers list --name <customer-name> --json

# Try filtering by customer ID
xbe view job-production-plans list --start-on <date> --customer <customer-id>

# If that returns no results, try searching by name
xbe view job-production-plans list --start-on <date> --q "<customer-name>"
```
