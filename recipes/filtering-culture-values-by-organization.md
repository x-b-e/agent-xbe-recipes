---
title: Filtering culture values by organization
when: When you need to retrieve all culture values for a specific organization, especially when unfiltered queries return incomplete results
---

# Filtering culture values by organization

## Problem
When listing culture values without filters, the API may return incomplete results (only showing some values instead of all values for an organization).

## Solution
Use the `--organization` filter with the proper format: `"<org-type>|<org-id>"`

```bash
# First, identify the organization ID and type
xbe view memberships list --user <user-id>

# Then filter culture values using the Type|ID format
xbe view culture-values list --organization "Broker|<organization-id>" --json
```

## Format Details
- Organization type must be capitalized (e.g., `Broker`, `Customer`)
- Use pipe character `|` to separate type from ID
- The organization type and ID can be found in membership or organization listings

## Sorting Results
Combine with jq to sort by sequence position:

```bash
xbe view culture-values list --organization "Broker|<organization-id>" --json | jq 'sort_by(.sequence_position)'
```

## Example Output Structure
Culture values include:
- `id`: Unique identifier
- `name`: Value name
- `description`: Detailed description
- `sequence_position`: Display order (0-indexed)
- `organization`: Organization name
- `organization_id`: Organization ID
- `organization_type`: Type (e.g., "brokers")
