---
title: Creating HTML visualizations from lane summary JSON output
when: When you need to create shareable HTML charts or visualizations from lane summary data for stakeholders
---

# Creating HTML visualizations from lane summary JSON output

## Context
The XBE CLI can output lane summaries in JSON format, which can be parsed and transformed into visual HTML reports for sharing with stakeholders.

## Workflow

### 1. Get lane summary data in JSON format
Use the `--json` flag on lane summary commands:

```bash
xbe summarize lane-summary create \
  --group-by origin,destination \
  --filter broker=<broker-id> \
  --filter year=<year> \
  --metrics tons_sum \
  --sort tons_sum:desc \
  --limit <limit> \
  --json
```

### 2. Parse the JSON output
The output includes:
- `headers`: Array of column names
- `values`: Array of arrays with the data
- `rows`: Array of objects with named fields

Example structure:
```json
{
  "headers": ["origin_name", "destination_name", "tons_sum"],
  "values": [["Origin A", "Dest B", "12345.67"], ...],
  "rows": [
    {"origin_name": "Origin A", "destination_name": "Dest B", "tons_sum": "12345.67"},
    ...
  ]
}
```

### 3. Create HTML visualization
Use the `Write` tool to generate an HTML file with:
- Chart/table of the data
- Custom styling (can incorporate brand colors)
- Interactive elements (hover effects, etc.)
- Professional formatting for stakeholder consumption

## Tips
- Use `mkdir -p tmp` before writing to ensure the output directory exists
- The `rows` array is easiest to work with as it provides named fields
- Consider calculating percentages relative to the max value for bar chart widths
- HTML files can be opened directly in browsers or emailed to stakeholders
- Add contextual information in footers (data source, totals, date range, etc.)

## Related recipes
- analyzing-top-lanes-for-a-broker.md: Getting the underlying lane data
- creating-ascii-bar-charts-from-lane-summaries.md: Creating terminal-based visualizations
