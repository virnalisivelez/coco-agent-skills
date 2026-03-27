---
name: coco-business-ops
description: Business operations assistant for tracking revenue, expenses, clients, content, and goals. Use when the user needs to build a business dashboard, track revenue streams, manage a client pipeline, plan content calendars, set growth goals, or generate weekly business reviews. Covers the full operations stack for small businesses.
license: Apache-2.0
metadata:
  author: good-stories-llc
  version: "1.0"
  brand: COCO
  website: https://goodstories.world
---

# Business Operations Assistant

Help users build and manage business operations — revenue tracking, expense management, client pipelines, content calendars, growth goals, and weekly reviews. This skill provides the methodology and structure; the user provides their data.

## When to Use

- User wants to track business revenue or expenses
- User needs a client pipeline or CRM system
- User asks about content planning or editorial calendars
- User wants to set business goals and track progress
- User needs a weekly business review framework
- User asks to build a business dashboard

## Module 1: Revenue Tracking

Help users set up revenue tracking by source/stream:

```python
# Revenue entry structure
revenue_entry = {
    "date": "2026-03-27",
    "amount": 29.99,
    "stream": "Product Sales",  # e.g., Freelance, SaaS, Products, Consulting
    "notes": "Gumroad sale - Growth Command Center"
}
```

### Key Metrics to Track
- **Monthly recurring revenue (MRR):** Sum of subscription income
- **One-time revenue:** Product sales, project fees
- **Revenue by stream:** Which income sources are growing?
- **Month-over-month growth:** Trend direction

### Analysis Prompt
```
Analyze this revenue data and provide:
1. Total revenue this month vs last month (% change)
2. Top 3 revenue streams by volume
3. Streams trending up vs down
4. One specific recommendation to increase revenue next month

Revenue data: {revenue_entries}
```

## Module 2: Expense Tracking

Track costs by category to understand margins:

```python
expense_entry = {
    "date": "2026-03-27",
    "amount": 20.00,
    "category": "Tools",  # Tools, Ads, Contractors, Hosting, Marketing, Other
    "notes": "Claude API costs"
}
```

### Key Metrics
- **Monthly burn rate:** Total expenses this month
- **Net profit:** Revenue minus expenses
- **Cost by category:** Where is money going?
- **Profit margin:** Net profit / revenue as percentage

## Module 3: Client Pipeline

CRM-style tracking with stage progression:

```python
client_entry = {
    "name": "Acme Corp",
    "value": 5000,
    "stage": "Proposal",  # Lead → Contacted → Proposal → Won → Lost
    "notes": "Follow up next Tuesday"
}
```

### Pipeline Metrics
- **Total pipeline value:** Sum of all non-Lost deal values
- **Conversion rate:** Won / (Won + Lost)
- **Average deal size:** Mean value of Won deals
- **Stage distribution:** How many clients at each stage?

### Follow-Up Prompt
```
Review this client pipeline and suggest:
1. Which leads need follow-up this week (contacted > 5 days ago)
2. Which proposals are at risk of going cold
3. Patterns in won vs lost deals
4. Next best action for each active client

Pipeline: {client_entries}
```

## Module 4: Content Calendar

Plan and track content across platforms:

```python
content_entry = {
    "date": "2026-03-28",
    "platform": "LinkedIn",  # LinkedIn, Twitter/X, Email, Blog, YouTube
    "title": "How I track business ops with AI",
    "status": "Draft"  # Draft → Scheduled → Published
}
```

### Planning Framework
1. **Monday:** Plan the week's content themes
2. **Tuesday-Thursday:** Draft and schedule posts
3. **Friday:** Review engagement, note what worked
4. **Monthly:** Analyze which platforms and topics drive results

### Content Planning Prompt
```
Based on these recent topics and engagement data, suggest:
1. Three content ideas for next week
2. Which platform each idea fits best
3. Best posting times based on past engagement
4. One content experiment to try

Recent content: {content_entries}
Business goals: {goals}
```

## Module 5: Growth Goals

Set quarterly targets and track progress:

```python
goal_entry = {
    "name": "Hit $5k MRR",
    "target": 5000,
    "current": 2300,
    "type": "Revenue",  # Revenue, Clients, Content, Custom
    "quarter": "Q2 2026"
}
```

### Goal Types with Auto-Computation
- **Revenue goals:** `current` = sum of revenue entries in that quarter
- **Client goals:** `current` = count of pipeline entries not in "Lost" stage
- **Content goals:** `current` = count of published content in that quarter
- **Custom goals:** User manually updates `current` value

### Quarter Mapping
- Q1: January-March
- Q2: April-June
- Q3: July-September
- Q4: October-December

## Module 6: Weekly Review

Structured weekly reflection for continuous improvement:

### Weekly Review Template

```markdown
# Weekly Review — {date}

## Wins
- What went well this week?
- What revenue came in?
- What content performed?

## Challenges
- What didn't go as planned?
- What took longer than expected?
- What blocked progress?

## Lessons
- What did I learn?
- What would I do differently?
- What surprised me?

## Next Week's Focus
- Top 3 priorities for next week
- Key deadlines
- One experiment to run
```

### Review Analysis Prompt
```
Analyze my weekly reviews over the past month and identify:
1. Recurring wins (patterns of success to double down on)
2. Recurring challenges (systemic issues to address)
3. Lessons that should become permanent processes
4. Progress toward quarterly goals

Reviews: {review_entries}
Goals: {goal_entries}
```

## Putting It All Together: Dashboard KPIs

When building a dashboard view, compute these four KPIs:

1. **Net Revenue** = sum(revenue, current month) - sum(expenses, current month)
2. **Total Active Clients** = count(pipeline entries where stage != "Lost")
3. **Conversion Rate** = Won / (Won + Lost) from pipeline
4. **Content Published** = count(content entries with status "Published", current month)

## Storage Options

- **Local files:** JSON or SQLite for single-user, offline-first
- **Supabase:** Free tier gives you Postgres + auth + API
- **Google Sheets:** Simple but limited for automation
- **Notion:** Good UI but harder to automate

For AI-powered analysis, store data in a structured format (JSON/database) so you can pass it to LLM prompts for insights.
