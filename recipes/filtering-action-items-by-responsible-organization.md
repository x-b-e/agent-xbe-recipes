---
title: Filtering action items by responsible organization
when: When you need to filter action items by the organization the work is assigned to, such as finding XBE internal work vs customer-specific work
---

## Understanding Responsible Organization

The `responsible_org_name` field on action items indicates which organization the work is assigned to/for, NOT who is doing the work. XBE team members often work on items assigned to customer organizations.

## Filter Syntax

Use the `--responsible-organization` filter with the format `Type|ID`:

```bash
xbe view action-items list --responsible-organization "Broker|<broker-id>"
```

## Finding XBE Internal Work

To find action items that are internal XBE work (as opposed to customer-specific work):

```bash
# XBE Office has broker ID 49
xbe view action-items list --status in_progress --responsible-organization "Broker|49"
```

## Common Mistake

Don't use lowercase `brokers` - the API expects the capitalized type name:

```bash
# ❌ Wrong
xbe view action-items list --responsible-organization "brokers|49"

# ✅ Correct
xbe view action-items list --responsible-organization "Broker|49"
```

## Multiple Organizations

You can filter by multiple organizations using comma separation:

```bash
xbe view action-items list --responsible-organization "Broker|<broker-id-1>,Broker|<broker-id-2>"
```
