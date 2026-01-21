---
title: Specifying organization references in membership commands
when: When you need to specify an organization in membership create or update commands using the correct type|id format
---

# Specifying organization references in membership commands

When creating or updating memberships, you need to specify the organization using a specific reference format that includes both the organization type and ID.

## Organization Reference Format

The format is: `<OrganizationType>|<organization-id>`

Supported organization types:
- `Broker` - for broker organizations
- `Customer` - for customer organizations  
- `Trucker` - for trucker organizations
- `MaterialSupplier` - for material supplier organizations
- `Developer` - for developer organizations

## Examples

### Creating a membership with a broker organization

```bash
# First find the broker ID
xbe view brokers list --company-name "<company-name>"

# Then create the membership using Broker|<id> format
xbe do memberships create --user <user-id> --organization "Broker|<broker-id>" --kind manager
```

### Creating a membership with other organization types

```bash
# Customer organization
xbe do memberships create --user <user-id> --organization "Customer|<customer-id>" --kind operations

# Trucker organization
xbe do memberships create --user <user-id> --organization "Trucker|<trucker-id>" --kind manager

# Material supplier organization
xbe do memberships create --user <user-id> --organization "MaterialSupplier|<supplier-id>" --kind operations
```

## Notes

- The organization type is case-sensitive (use `Broker` not `broker`)
- The entire organization reference should be quoted when passed as a command-line argument
- You typically need to look up the organization ID first using the appropriate `xbe view` command before creating the membership
