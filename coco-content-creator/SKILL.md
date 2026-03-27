---
name: coco-content-creator
description: Generate multi-platform content (LinkedIn, Twitter/X, email, blog) from a single input. Use when the user needs to create social media posts, email campaigns, blog drafts, or repurpose content across platforms. Handles platform-specific formatting, tone adaptation, and content calendaring.
license: Apache-2.0
metadata:
  author: good-stories-llc
  version: "1.0"
  brand: COCO
  website: https://goodstories.world
---

# Multi-Platform Content Creator

Generate platform-specific content from a single idea, article, or topic. One input becomes LinkedIn posts, tweets, email drafts, and blog outlines — each adapted to the platform's tone, format, and audience expectations.

## When to Use

- User wants to create content for multiple social platforms at once
- User needs to repurpose an article or idea across channels
- User asks for LinkedIn posts, tweets, email sequences, or blog drafts
- User wants a content calendar or batch content generation

## Platform Specifications

Each platform has distinct requirements:

| Platform | Tone | Length | Format |
|----------|------|--------|--------|
| LinkedIn | Professional, insight-driven | 150-300 words | Hook line, body, CTA, 3-5 hashtags |
| Twitter/X | Sharp, concise, conversational | 280 chars max (or thread) | Hook + insight, optional thread |
| Email | Personal, direct, value-first | 200-400 words | Subject line, body, single CTA |
| Blog | In-depth, SEO-aware | 800-1500 words | H2 sections, intro, conclusion |

## Step 1: Define the Content Source

Start with a single input — an idea, article summary, product launch, or lesson learned:

```python
content_input = {
    "topic": "How I automated my daily news briefing with AI",
    "key_points": [
        "Built an RSS scraper that feeds into Claude",
        "Saves 45 minutes every morning",
        "Cost: $0.02/day in API calls"
    ],
    "audience": "Builders and creators using AI tools",
    "cta": "Link to the tool or product"
}
```

## Step 2: Generate Platform-Specific Content

### LinkedIn Post Generator

```python
LINKEDIN_PROMPT = """Write a LinkedIn post about this topic. Follow this structure:

1. Hook (first line — must stop the scroll, no clickbait)
2. Story or insight (2-3 short paragraphs, use line breaks between them)
3. Key takeaway (one clear lesson)
4. Call to action (soft, not salesy)
5. 3-5 relevant hashtags

Topic: {topic}
Key points: {key_points}
Audience: {audience}
CTA: {cta}

Rules:
- Write in first person
- No corporate jargon
- Short paragraphs (1-2 sentences each)
- Use line breaks generously (LinkedIn rewards white space)
- The hook must work WITHOUT the rest of the post"""
```

### Twitter/X Thread Generator

```python
TWITTER_PROMPT = """Write a Twitter/X thread about this topic. Structure:

Tweet 1 (Hook): Bold claim or surprising stat. Must work standalone. Under 280 chars.
Tweets 2-5: One key point per tweet. Concrete, specific, no filler.
Final tweet: Takeaway + soft CTA.

Topic: {topic}
Key points: {key_points}

Rules:
- Each tweet must be under 280 characters
- Number tweets (1/, 2/, etc.)
- No hashtags in individual tweets (add to final tweet only)
- Conversational tone, not formal
- Each tweet should be interesting on its own"""
```

### Email Draft Generator

```python
EMAIL_PROMPT = """Write a short email about this topic. Structure:

Subject line: Clear, specific, creates curiosity (under 50 chars)
Opening: Personal, direct, no "I hope this finds you well"
Body: One key insight or story (2-3 short paragraphs)
CTA: Single clear action (reply, click, try something)
Sign-off: Brief, warm

Topic: {topic}
Key points: {key_points}
Audience: {audience}
CTA: {cta}

Rules:
- Write like you're emailing one person, not a list
- Subject line should work in a mobile preview (first 35 chars matter)
- No multiple CTAs — one action only
- Under 300 words total"""
```

### Blog Outline Generator

```python
BLOG_PROMPT = """Create a blog post outline about this topic. Structure:

Title: SEO-friendly, includes primary keyword, under 60 chars
Meta description: Under 155 chars, includes keyword
H2 sections: 4-6 sections, each with 2-3 bullet points of what to cover
Intro: What the reader will learn and why it matters (2-3 sentences)
Conclusion: Key takeaway + CTA

Topic: {topic}
Key points: {key_points}
Audience: {audience}

Rules:
- Title should match what someone would Google
- Each H2 should be a question or clear topic
- Include where to add examples, screenshots, or data"""
```

## Step 3: Batch Generate All Platforms

```python
def generate_all_content(content_input: dict) -> dict:
    results = {}
    platforms = {
        "linkedin": LINKEDIN_PROMPT,
        "twitter": TWITTER_PROMPT,
        "email": EMAIL_PROMPT,
        "blog": BLOG_PROMPT,
    }
    for platform, prompt_template in platforms.items():
        prompt = prompt_template.format(**content_input)
        results[platform] = call_llm(
            system="You are a skilled content strategist.",
            user=prompt,
        )
    return results
```

## Step 4: Review and Adapt

After generating, review each piece:

1. **LinkedIn:** Does the hook stop scrolling? Is the CTA natural?
2. **Twitter:** Is each tweet under 280 chars? Does tweet 1 work alone?
3. **Email:** Is the subject line compelling in 35 chars?
4. **Blog:** Are the H2s what someone would Google?

## Voice Profile (Optional)

For consistent brand voice, define your voice profile:

```python
VOICE_PROFILE = """
Tone: Direct, practical, no fluff
Perspective: Builder who shares real experiences
Avoid: Corporate jargon, "leverage", "synergy", buzzwords
Prefer: Short sentences, concrete examples, specific numbers
Audience relationship: Peer-to-peer (not guru-to-student)
"""
```

Add `VOICE_PROFILE` to each prompt's system message for consistency.

## Content Calendar Integration

To plan content over time:

```python
def plan_content_week(topics: list[str], platforms: list[str]) -> list[dict]:
    """Generate a week of content from a list of topics."""
    import datetime
    calendar = []
    for i, topic in enumerate(topics):
        day = datetime.date.today() + datetime.timedelta(days=i)
        platform = platforms[i % len(platforms)]
        calendar.append({
            "date": day.isoformat(),
            "platform": platform,
            "topic": topic,
            "status": "draft"
        })
    return calendar
```

## Dependencies

Your LLM provider SDK (anthropic, openai, httpx for OpenRouter, etc.)
