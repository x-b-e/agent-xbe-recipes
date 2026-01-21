---
title: Creating broker memberships with organization parameter format
when: When you need to create a membership and must specify the organization using the correct parameter format (Type|ID)
---

# Creating broker memberships with organization parameter format

When creating memberships, the `--organization` parameter requires a specific format combining the organization type and ID.

## Workflow

1. Find the organization ID (e.g., for a broker):
```bash
xbe view brokers list --company-name "<company-name>"
```

2. Create the membership using the `Type|ID` format:
```bash
xbe do memberships create --user <user-id> --organization "Broker|<broker-id>" --kind <kind>
```

## Key details

- The organization parameter format is: `"<Type>|<ID>"`
- Type must be capitalized (e.g., "Broker", not "broker")
- The entire value should be quoted
- Common types: Broker, Customer, Trucker
- Kind can be: `operations` or `manager`

## Example

Creating a manager membership for user 1 at broker 297:
```bash
xbe do memberships create --user 1 --organization "Broker|297" --kind manager
```
