---
title: Filtering job production plans by multiple customers
when: When you need to find job production plans for multiple related customers (e.g., different entities within the same group like TrueRock Construction, TrueRock Materials, etc.)
---

# Filtering job production plans by multiple customers

When searching for job production plans for a company group (e.g., "TrueRock", "ABC Companies"), first identify all related customer entities, then filter by all their IDs.

## Step 1: Find all related customer entities

```bash
xbe view customers list --name <company-name> --json
```

This will return all customers matching the name, which may include multiple entities within the same corporate group.

## Step 2: Extract customer IDs

From the JSON output, note all the customer IDs. For example, a search for "TrueRock" might return:
- TrueRock Construction, Inc. (ID: <id-1>)
- TrueRock Materials Inc. (ID: <id-2>)
- TrueRock Transport, Inc. (ID: <id-3>)
- TrueRock Group Equipment Movement (ID: <id-4>)

## Step 3: Filter plans by multiple customer IDs

```bash
xbe view job-production-plans list --start-on <date> --customer <id-1>,<id-2>,<id-3>,<id-4>
```

The `--customer` flag accepts comma-separated customer IDs to include plans for all specified customers.

## Notes

- The customer filter uses exact ID matching, so you must include all related entity IDs to get complete results for a company group
- Don't assume a company name maps to a single customer - corporate groups often have multiple legal entities in the system
- Use `--json` flag in step 1 to make it easier to parse and extract IDs programmatically
