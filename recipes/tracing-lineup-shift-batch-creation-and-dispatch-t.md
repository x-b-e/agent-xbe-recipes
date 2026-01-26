---
title: Tracing lineup shift batch creation and dispatch timeline patterns
when: When you need to understand the complete timeline of how lineup shifts were created and dispatched in batches, including identifying manual entry phases, batch patterns, dispatch automation timing, and fulfillment speed
---

# Tracing lineup shift batch creation and dispatch timeline patterns

This recipe shows how to trace the complete timeline of lineup shifts from initial creation through batch dispatch, revealing manual entry patterns, timing phases, and automated dispatch sequences.

## Overview

When analyzing lineup operations, you may need to understand:
- How long did it take to create all the shifts?
- Were shifts entered in batches or continuously?
- What was the gap between creation and dispatch?
- How quickly did the dispatch automation work?
- How fast were shifts fulfilled after dispatch?

## Step 1: Get the lineup and its shifts

First, identify the lineup and retrieve all its shifts with timestamps:

```bash
# Find the lineup
xbe view lineups show <lineup-id> --json

# Get all shifts for this lineup with creation timestamps
xbe view lineup-shifts list --lineup <lineup-id> --json --limit 1000 \
  --fields id,trucker-id,created-at \
  --sort created-at
```

Key fields to examine:
- `created-at`: When each shift was added to the lineup
- `trucker-id`: Shows if shifts were grouped by trucker during entry

## Step 2: Analyze shift creation patterns

Process the shifts to identify timing patterns:

```bash
# Extract and analyze creation timestamps
xbe view lineup-shifts list --lineup <lineup-id> --json --limit 1000 \
  --fields id,trucker,created-at --sort created-at | \
  jq -r '.[] | "\(."created-at") - \(.trucker."company-name")"'
```

Look for:
- **Clusters**: Multiple shifts created within seconds (batch entry)
- **Gaps**: Pauses between entry sessions
- **Patterns**: Are shifts grouped by trucker? (suggests working through a list)
- **Speed**: Rapid entries (0.2-2 seconds apart) vs slower manual entry

## Step 3: Get dispatch events and timing

Retrieve the dispatch records to see when shifts were dispatched:

```bash
# Get lineup dispatches
xbe view lineup-dispatches list --lineup <lineup-id> --json --limit 1000 \
  --fields id,created-at --sort created-at

# Get individual dispatch shift records with timestamps
xbe view lineup-dispatches show <lineup-dispatch-id> --json | \
  jq '."dispatch-shifts"[] | {id, "created-at", trucker: .trucker."company-name"}'
```

Key insights:
- Time between last shift created and dispatch triggered
- How many shifts were dispatched in the batch
- Duration of the dispatch creation process (first to last dispatch-shift)

## Step 4: Analyze fulfillment timing

Check how quickly shifts were fulfilled after dispatch:

```bash
# Get dispatch shifts with fulfillment timestamps
xbe view dispatch-shifts list --dispatch <lineup-dispatch-id> --json --limit 1000 \
  --fields id,created-at,fulfilled-at,trucker --sort fulfilled-at
```

Calculate:
- Time from dispatch creation to first fulfillment
- Time from dispatch creation to last fulfillment
- Total fulfillment rate (all shifts fulfilled in X seconds)

## Step 5: Create a timeline visualization

Process the data to create a comprehensive timeline:

```bash
# Combine data into a timeline format
cat > timeline.txt << 'EOF'
DETAILED TIMELINE

PHASE 1: SHIFT CREATION
[Manual entry phases with timestamps]

PHASE 2: DISPATCH
[Dispatch trigger time and automation speed]

PHASE 3: FULFILLMENT
[Acceptance times and completion rate]

SUMMARY METRICS:
- Total creation time: [first to last shift]
- Creation phases: [number of distinct entry sessions]
- Average entry speed: [shifts per minute]
- Dispatch lag: [time between last entry and dispatch]
- Dispatch automation: [time to create all dispatch records]
- Fulfillment speed: [time to complete all acceptances]
EOF
```

## Example Pattern Analysis

Typical patterns you might discover:

**Batch Entry Pattern**:
- Shifts entered in clusters by trucking company
- 3-5 shifts per company in 0.5-2 seconds
- Suggests dispatcher working through a pre-planned list

**Multi-Session Entry**:
- Session 1: Morning (11 minutes, 57 shifts)
- Gap: Break or other work
- Session 2: Afternoon (7 minutes, 22 shifts)
- Session 3: Final batch (3 minutes, 28 shifts)

**Automated Dispatch**:
- Single button click triggers cascade
- System creates 50 dispatch records in ~200ms
- Shows high automation efficiency

**Rapid Fulfillment**:
- Sub-5-second fulfillment times
- Suggests pre-arranged assignments (auto-tender)
- Truckers were expecting the dispatch

## Key Insights This Reveals

1. **Manual vs Automated Work**: Distinguishes slow manual entry (hours) from fast automated dispatch (milliseconds)

2. **Operational Patterns**: Reveals how dispatchers organize their work (by trucker, by time window, etc.)

3. **System Performance**: Shows automation efficiency and fulfillment speed

4. **Pre-Assignment Model**: Fast fulfillment suggests shifts are pre-arranged before dispatch, making dispatch a notification rather than assignment

## Related Workflows

- For individual shift tracking: See "Tracing individual lineup shift lifecycle from creation to fulfillment"
- For dispatcher identification: See "Tracing lineup dispatch timeline and dispatcher activity"
- For understanding data model: See "Understanding lineup-dispatch data model limitations"
