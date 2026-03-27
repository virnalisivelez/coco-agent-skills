---
name: coco-marketing-planner
description: Generate complete marketing campaigns from product details. Use when planning launches, creating content strategies, building campaign briefs, or generating messaging and channel recommendations from a product name, target audience, and budget.
license: Apache-2.0
metadata:
  author: good-stories-llc
  version: "1.0"
  brand: COCO
---

# Marketing Planner

Generate complete, actionable marketing campaigns from product details. Takes a product name, target audience, and budget as input and produces messaging, content ideas, channel strategy, and metrics -- ready to execute.

## When to Use

- User wants to plan a product launch campaign
- User needs content strategy recommendations for a product
- User asks for messaging angles, channel selection, or campaign structure
- User wants a campaign brief with metrics and budget allocation

## Input Requirements

Gather these three inputs before generating a campaign:

1. **Product name and description:** What is being sold and what problem it solves
2. **Target audience:** Who buys this (demographics, psychographics, pain points)
3. **Budget:** Monthly or campaign budget in dollars (or "organic only" for zero budget)

Optional context that improves output:
- Existing brand voice/tone guidelines
- Competitor landscape
- Timeline (launch date, campaign duration)
- Existing audience size (email list, social followers)

## Campaign Generation Process

### Step 1: Define the Messaging Angle

The messaging angle is the single most important element. It answers: "Why should someone care about this product right now?"

#### Angle Framework

Choose one primary angle from these proven patterns:

| Angle | When to Use | Example |
|---|---|---|
| **Problem-Solution** | Audience has a known pain point | "Stop spending 3 hours a week on reports. Generate them in 5 minutes." |
| **Transformation** | Product enables a meaningful change | "Go from scattered notes to a structured thesis in weeks, not months." |
| **Contrast** | Competing with established alternatives | "Unlike [category], this does X without Y." |
| **Social Proof** | You have impressive traction data | "Join 2,000+ professionals who switched this quarter." |
| **Urgency** | Limited availability or time-sensitive | "Launch pricing ends Friday. 40% off the first 100 licenses." |
| **Identity** | Audience identifies with a role | "Built for operators who build and ship, not just plan." |

#### Messaging Hierarchy

1. **Headline:** One sentence that captures attention (the angle)
2. **Subhead:** One sentence that explains what the product does
3. **Proof point:** One data point or credential that builds trust
4. **Call to action:** One clear next step

Example:
```
Headline:    Your next campaign, planned in 10 minutes.
Subhead:     AI-powered campaign briefs with messaging, channels, and metrics.
Proof point: Used by 500+ marketing teams since launch.
CTA:         Start your first campaign -- free.
```

### Step 2: Generate 3 Content Ideas

Each content idea should be specific enough to execute immediately.

#### Content Idea Template

```
Title: [Specific, attention-grabbing title]
Format: [Blog post / Video / Twitter thread / LinkedIn post / Email / Infographic]
Hook: [First line or opening that stops the scroll]
Outline:
  1. [Section or point 1]
  2. [Section or point 2]
  3. [Section or point 3]
CTA: [What the reader should do after consuming]
Rationale: [Why this content works for the target audience]
```

#### Content Type Selection

| Audience Behavior | Best Content Types |
|---|---|
| Researching solutions | Long-form blog, comparison guides, case studies |
| Active on social media | Short video, Twitter threads, LinkedIn posts |
| Already aware of product | Email sequences, product demos, testimonials |
| Technical decision-makers | Technical deep-dives, architecture docs, benchmarks |
| Budget decision-makers | ROI calculators, case studies, executive summaries |

### Step 3: Recommend Channels

Select 2-3 channels based on where the target audience already spends time.

#### Channel Selection Matrix

| Channel | Best For | Cost Profile | Time to Results |
|---|---|---|---|
| **Organic Twitter/X** | Tech, developer, AI audiences | Free (time only) | 2-4 weeks |
| **LinkedIn organic** | B2B, professional audiences | Free (time only) | 2-4 weeks |
| **LinkedIn ads** | B2B lead generation | $5-15 per click | 1-2 weeks |
| **Google Search ads** | High-intent buyers searching for solutions | $1-10 per click | Days |
| **Email marketing** | Existing audience nurturing | Low (tool cost) | Immediate |
| **YouTube** | Tutorial/educational content | Free (time) or ad spend | 4-8 weeks |
| **Reddit** | Niche communities, technical audiences | Free or low ad spend | 1-2 weeks |
| **Product Hunt** | Product launches, tech-forward audiences | Free | Launch day |
| **SEO/Content** | Long-term organic traffic | Free (time only) | 3-6 months |
| **Partnerships** | Audience crossover with complementary products | Revenue share | 2-4 weeks |

#### Channel Recommendation Format

For each recommended channel, provide:

```
Channel: [Name]
Why: [Specific reason this channel fits the audience]
Tactic: [Exact approach -- not "post content" but "publish 3x/week:
         1 educational thread, 1 product tip, 1 audience question"]
Budget allocation: [$ or % of total budget]
Expected result: [Specific metric target]
Timeline: [When to expect results]
```

### Step 4: Define Metrics

Every campaign needs measurable targets. Define metrics at three levels:

#### Awareness Metrics (top of funnel)
- Impressions / reach
- Website visits from campaign sources
- Social media engagement rate (likes, shares, comments)

#### Consideration Metrics (middle of funnel)
- Email signups / lead captures
- Content downloads
- Demo requests or trial starts
- Time on page for key content

#### Conversion Metrics (bottom of funnel)
- Purchases / revenue
- Cost per acquisition (CPA)
- Conversion rate (visits to purchase)
- Return on ad spend (ROAS)

#### Setting Targets

Use these benchmarks as starting points (adjust based on industry and audience):

| Metric | Benchmark Range |
|---|---|
| Email open rate | 20-30% |
| Email click rate | 2-5% |
| Landing page conversion | 2-5% for cold traffic, 10-20% for warm |
| Social engagement rate | 1-3% |
| Google Ads CTR | 2-5% |
| LinkedIn Ads CTR | 0.4-1% |
| Cost per email subscriber | $1-5 |
| Cost per purchase (digital products) | $5-30 |

### Step 5: Budget Allocation

#### Zero Budget (Organic Only)

Allocate time instead of money:

```
60% - Content creation (blog posts, social content, email)
20% - Community engagement (comments, forums, groups)
10% - Partnerships and cross-promotion
10% - Analytics and optimization
```

#### Under $500/month

```
40% - Paid social ads (targeted at highest-intent audience)
30% - Content creation (quality over quantity)
20% - Email marketing tools
10% - Analytics tools
```

#### $500-$2000/month

```
35% - Paid acquisition (Google/LinkedIn ads)
25% - Content creation and distribution
20% - Email marketing and automation
10% - Retargeting ads
10% - Testing and experimentation
```

## Campaign Brief Template

Deliver the complete campaign in this format:

```markdown
# Campaign Brief: [Product Name]

## Product
[1-2 sentences: what it is, who it serves, what problem it solves]

## Target Audience
- Demographics: [age, role, industry]
- Pain points: [top 3 frustrations this product addresses]
- Where they are: [platforms, communities, content they consume]

## Messaging

**Primary angle:** [chosen angle type]

**Headline:** [attention-grabbing headline]
**Subhead:** [explanatory subhead]
**Proof point:** [trust builder]
**CTA:** [clear next step]

## Content Plan

### Idea 1: [Title]
[Full content idea with format, hook, outline, CTA, rationale]

### Idea 2: [Title]
[Full content idea]

### Idea 3: [Title]
[Full content idea]

## Channel Strategy

### Channel 1: [Name]
[Why, tactic, budget, expected result, timeline]

### Channel 2: [Name]
[Why, tactic, budget, expected result, timeline]

### Channel 3: [Name]
[Why, tactic, budget, expected result, timeline]

## Metrics and Targets

| Metric | Target | Measurement Method |
|---|---|---|
| [metric] | [target] | [how to measure] |

## Budget Allocation

| Category | Amount | % of Total |
|---|---|---|
| [category] | $X | X% |

## Timeline

| Week | Activities |
|---|---|
| Week 1 | [setup, content creation] |
| Week 2 | [launch, first distribution] |
| Week 3 | [optimization, second wave] |
| Week 4 | [review, iterate] |
```

## Output Expectations

1. Deliver a complete campaign brief using the template above
2. All recommendations must be specific and actionable -- no vague advice
3. Budget allocations must sum to 100%
4. Every channel recommendation includes a concrete tactic, not just the channel name
5. Metrics include realistic targets based on the benchmarks provided
