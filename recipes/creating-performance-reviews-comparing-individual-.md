---
title: Creating performance reviews comparing individual against team benchmarks
when: When you need to create a comprehensive performance review for an individual by comparing their metrics against team averages and identifying mentorship opportunities based on complementary strengths and weaknesses
---

# Creating performance reviews comparing individual against team benchmarks

This recipe demonstrates how to create comprehensive performance reviews by comparing an individual's performance against team statistics to identify relative strengths, weaknesses, and mentorship opportunities.

## Overview

Performance reviews are most meaningful when they show:
1. **Individual metrics** - Raw performance data for the person being reviewed
2. **Comparative ranking** - Where they stand relative to peers
3. **Statistical context** - Team averages, standard deviations, and distributions
4. **Mentorship matching** - Who can mentor them (in weakness areas) and who they can mentor (in strength areas)

## Step-by-step workflow

### 1. Gather individual performance data

First, collect the subject's data for the review period:

```bash
# Example: Get job production plans for specific PM in target month
xbe view job-production-plans list \
  --json \
  --start-date <start-date> \
  --end-date <end-date> \
  --project-manager-id <pm-id> \
  --per-page 1000
```

### 2. Gather comparative team data

Collect the same metrics for all team members:

```bash
# Get all PMs' data for the same period
xbe view job-production-plans list \
  --json \
  --start-date <start-date> \
  --end-date <end-date> \
  --per-page 1000
```

### 3. Calculate key metrics for comparison

For each person (including the subject), calculate:

**Volume metrics:**
- Total tonnage/volume handled
- Number of plans managed
- Average plan size
- Number of unique projects

**Performance metrics:**
- Goal achievement rate (actual vs planned)
- Standard deviation of achievement (consistency)
- Percentage of plans exceeding 100%
- Percentage of plans below 80%

**Operational metrics:**
- Cancellation rate
- Surplus variance (approved vs actual)
- Any domain-specific KPIs

### 4. Calculate team statistics

For each metric, calculate:
- Team average
- Team median
- Standard deviation
- Min/max values
- Percentile rankings

### 5. Identify relative strengths

A strength is where the individual:
- Ranks in top 3 on the team
- Exceeds team average by >10%
- Shows best-in-class performance on important metrics

For each strength:
1. State the ranking ("1st on team", "2nd best")
2. Provide the specific metric with comparison
3. Give concrete examples from the data
4. Explain business impact

### 6. Identify development areas

A development area is where the individual:
- Ranks in bottom 3 on the team
- Falls below team average by >10%
- Shows concerning patterns (high variance, outliers)

For each weakness:
1. State the ranking and metric
2. Compare to top performers
3. Provide specific examples of underperformance
4. Suggest root cause investigation questions

### 7. Create mentorship pairings

**Who should mentor the subject:**
For each development area, identify peers who:
- Excel in that specific metric (top 3)
- Have complementary strengths
- Could teach specific techniques

**Who the subject should mentor:**
For each strength area, identify peers who:
- Struggle in that metric (bottom performers)
- Could benefit from the subject's expertise
- Have receptive learning styles

### 8. Format discussion questions

Structure the review conversation:

**Opening (Positive):**
- Start with their #1 strength
- Ask them to explain their process
- Reinforce pride in top performance

**Development (Constructive):**
- Present data on weakness areas
- Ask open-ended questions about specific incidents
- Explore root causes collaboratively

**Growth & Mentorship:**
- Suggest specific peer mentors
- Propose concrete learning activities
- Offer to mentor others in strength areas

### 9. Create action plan

Provide:
- **Immediate actions** (this quarter)
- **Short-term goals** (next quarter) with specific targets
- **Long-term development** (6-12 months)

Each action should:
- Be specific and measurable
- Have a clear owner
- Reference the comparative data that motivated it

## Example output structure

```
================================================================================
<NAME> - <PERIOD> PERFORMANCE REVIEW
================================================================================

OVERVIEW
--------
[Summary of volume, projects, key numbers]

COMPARATIVE RANKING (among <N> team members)
---------------------------------------------
Metric 1: Xth (value) - context
Metric 2: Xth (value) - context
[...]

================================================================================
STRENGTHS - WHAT <NAME> IS GREAT AT
================================================================================

1. <STRENGTH NAME> (★★★★★ Ranking)
-----------------------------------
METRIC: <value> (vs team avg <value>)

What This Means:
[Plain English explanation of business impact]

Specific Examples to Discuss:
• <Date/project>: <specific data point>
• <Date/project>: <specific data point>

Impact: [Why this matters to the business]

[Repeat for each strength...]

================================================================================
DEVELOPMENT AREAS - WHERE <NAME> NEEDS WORK
================================================================================

1. <WEAKNESS NAME> (★★☆☆☆ Ranking - Needs Attention)
-----------------------------------------------------
METRIC: <value>
TEAM COMPARISON: Top performer <value>, Team Avg <value>

What This Means:
[Plain English explanation]

Specific Examples to Discuss:
• <Date/project>: <specific incident>

Questions to Explore:
- <Open-ended question>
- <Root cause question>

Action Plan:
[Specific recommendation with peer reference]

[Repeat for each weakness...]

================================================================================
MENTORSHIP OPPORTUNITIES
================================================================================

WHO SHOULD MENTOR <NAME>?
-------------------------

1. <PEER NAME> - <Skill Area>
   Why: [Specific metric comparison]
   What They Learn: [Concrete skills/techniques]

[...]

WHO COULD <NAME> MENTOR?
------------------------

1. <PEER NAME> - <Skill Area>
   Why: [Specific metric comparison]
   What They Learn: [Concrete skills/techniques]

[...]

================================================================================
DISCUSSION QUESTIONS FOR YOUR MEETING
================================================================================

Opening (Positive):
1. "[Question highlighting their #1 strength]"
2. "[Question about their process/approach]"

Development (Constructive):
3. "[Question about specific incident in weakness area]"
4. "[Question exploring root causes]"

Growth & Mentorship:
5. "[Suggestion to learn from peer]"
6. "[Invitation to mentor others]"

================================================================================
RECOMMENDED ACTIONS
================================================================================

Immediate (This Quarter):
[ ] <Specific action with metric target>
[ ] <Specific action with peer reference>

Short-term (Next Quarter):
[ ] <Goal with measurable target>
[ ] <Development activity>

Long-term (6-12 months):
[ ] <Strategic positioning>
[ ] <Leadership development>
```

## Key principles

1. **Data-driven**: Every strength and weakness backed by specific metrics and rankings
2. **Comparative**: Always show where they stand relative to peers
3. **Actionable**: Specific examples, not generalities
4. **Balanced**: Start with strengths, address weaknesses constructively
5. **Collaborative**: Frame as growth opportunities with peer support
6. **Forward-looking**: End with concrete action plan and timeline

## Common pitfalls

- **Avoid**: Vague feedback like "needs improvement"
- **Instead**: "Ranks 8th of 9 with 18.2% cancellation rate vs team best of 0%"

- **Avoid**: Feedback without examples
- **Instead**: "Oct 27-29: three consecutive cancelled plans totaling 6,840 tons"

- **Avoid**: Weakness without solutions
- **Instead**: "Learn from Ben Yoder (0% cancellations) - shadow his pre-planning process"

- **Avoid**: One-sided reviews
- **Instead**: Identify strengths where they can mentor others, creating mutual value
