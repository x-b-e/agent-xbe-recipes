---
title: Creating memberships
when: When you need to create a new membership associating a user with an organization
---

# Creating memberships

To create a membership, use the `xbe do memberships create` command with required flags:

```bash
xbe do memberships create --user <user-id> --organization "<org-type>|<org-id>" --kind <kind>
```

**Important:** The `--organization` value must be quoted because it contains a pipe character that would otherwise be interpreted by the shell.

## Required flags

- `--user <user-id>`: The user ID
- `--organization "<Type>|<ID>"`: Organization in Type|ID format (must be quoted)
  - Valid types: Broker, Customer, Trucker, MaterialSupplier, Developer
- `--kind <kind>`: Role type - either `operations` (default) or `manager`

## Optional flags

- `--is-admin <true|false>`: Grant admin privileges
- `--title <title>`: Job title within the organization
- `--is-rate-editor <true|false>`: Allow editing rates
- `--is-time-card-auditor <true|false>`: Allow auditing time cards
- Various notification and permission flags (see `--help` for complete list)

## Example

```bash
# Create a manager membership at a broker
xbe do memberships create --user <user-id> --organization "Broker|<broker-id>" --kind manager

# Create an operations role with admin privileges
xbe do memberships create --user <user-id> --organization "Trucker|<trucker-id>" --kind operations --is-admin true
```

## Output

The command returns details about the created membership including:
- Membership ID
- User and organization details
- Role and permissions
- Notification settings
