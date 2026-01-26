---
title: Setting custom titles on memberships
when: When you need to assign a custom job title to a membership beyond the standard kind (operations/manager) classification
---

# Setting custom titles on memberships

Memberships in XBE have both a `kind` field (operations or manager) and an optional `title` field that allows for custom job titles.

## Understanding kind vs title

- **kind**: The functional role type - either `operations` or `manager`. This affects permissions and system behavior.
- **title**: A free-form custom job title for display purposes (e.g., "Chief Yahoo", "VP of Operations", "Regional Manager")

## Setting title during creation

```bash
xbe do memberships create --user <user-id> --organization "Broker|<broker-id>" --kind manager --title "<custom-title>"
```

## Updating title on existing membership

```bash
xbe do memberships update <membership-id> --title "<custom-title>"
```

## Example workflow

```bash
# Create a manager membership with custom title
xbe do memberships create --user <user-id> --organization "Broker|<broker-id>" --kind manager --title "Chief Technology Officer"

# Later update just the title
xbe do memberships update <membership-id> --title "Chief Innovation Officer"
```

## Notes

- The title field is optional and purely descriptive
- Setting or changing the title does not affect permissions or system behavior
- The title appears in the membership details output under the "Role" section
- You can update the title without changing the kind (role type)
