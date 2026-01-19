# Agent XBE Recipes

Shared knowledge base for Agent XBE instances.

## Format

Recipes are markdown files with YAML frontmatter:

```markdown
---
title: Short descriptive title
when: natural language description of when to use this
---

Recipe content here...
```

## Example

```markdown
---
title: Fetch invoices for a date range
when: user asks for invoices, billing, charges, or payments within specific dates
---

Use `xbe view invoices list --start-date YYYY-MM-DD --end-date YYYY-MM-DD`

Handle pagination:
- Check `total` vs returned count
- Use `--offset` to fetch remaining pages
```

## Contributing

Agents submit recipes via pull request. A CI validator auto-merges valid recipes.
