---
title: Creating HTML visualization reports from lane summaries
when: When you need to create a professional, interactive HTML report visualizing lane summary data with charts, statistics, and visual elements
---

# Creating HTML visualization reports from lane summaries

## Overview
You can create rich HTML reports from lane summary data by combining the CLI output with HTML/CSS templates. This is useful for creating shareable, visually appealing reports.

## Basic Pattern

### Step 1: Get the data from lane summary
```bash
xbe summarize lane-summary create \
  --filter broker=<broker-id> \
  --filter date_min=<start-date> \
  --filter date_max=<end-date> \
  --group-by origin,destination,trucker \
  --metrics tons_sum \
  --sort tons_sum:desc \
  --limit <limit>
```

### Step 2: Create HTML file with embedded data
Use the `Write` tool to create an HTML file that includes:
- The data embedded directly in the HTML structure
- CSS styling for visualization (progress bars, charts, cards)
- Interactive elements (hover effects, animations)
- Calculated statistics (totals, percentages, rankings)

### Step 3: Key HTML components to include

**Summary statistics section:**
- Total lanes analyzed
- Total volume across all lanes
- Top lane volume
- Number of active truckers/suppliers

**Visual elements:**
- Progress bars showing relative volumes (calculate percentage of max)
- Ranked items with visual badges
- Color-coded truckers/categories
- Hover effects for interactivity

**Data presentation:**
- Each lane as a separate item/card
- Origin → Destination route display
- Trucker badges or labels
- Tonnage prominently displayed

## Example Structure
```html
<!DOCTYPE html>
<html>
<head>
    <title>Lane Volume Analytics</title>
    <style>
        /* Styling for cards, progress bars, hover effects */
    </style>
</head>
<body>
    <header>
        <h1>Broker Name - Lane Analytics</h1>
    </header>
    
    <div class="stats-grid">
        <!-- Summary statistics cards -->
    </div>
    
    <div class="chart-container">
        <!-- Individual lane items with progress bars -->
        <div class="lane-item">
            <div class="lane-header">
                <span class="rank">1</span>
                <span class="route">Origin → Destination</span>
                <span class="tons">XX,XXX tons</span>
            </div>
            <div class="trucker">Trucker Name</div>
            <div class="progress-bar">
                <div class="progress-fill" style="width: XX%"></div>
            </div>
        </div>
    </div>
</body>
</html>
```

## Tips
- Calculate percentages for progress bars by dividing each value by the maximum value
- Use CSS gradients and animations for visual appeal
- Include responsive design for mobile viewing
- Add a timestamp/date range in the footer
- Consider using glass-morphism or modern design trends for professional appearance
- Highlight the top performer with special styling (animation, glow effects)

## Output
The HTML file can be opened directly in a browser and provides an interactive, shareable report that stakeholders can view without needing CLI access.
