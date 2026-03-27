# COCO Agent Skills

> Tools for humans building and creating with AI

Open-source [Agent Skills](https://agentskills.io) for business operations, intelligence briefings, and multi-platform content creation. Compatible with **32+ AI coding tools** including Claude Code, Cursor, VS Code Copilot, Gemini CLI, OpenAI Codex, and more.

## Skills

| Skill | Description |
|-------|-------------|
| [coco-intel-briefing](coco-intel-briefing/) | Build a daily intelligence briefing pipeline from RSS feeds |
| [coco-content-creator](coco-content-creator/) | Generate multi-platform content (LinkedIn, Twitter/X, email, blog) from a single input |
| [coco-business-ops](coco-business-ops/) | Business operations assistant — revenue, expenses, clients, content, goals |

## Quick Install

### Claude Code / Cursor / VS Code Copilot

```bash
# Copy all skills
git clone https://github.com/virnalisivelez/coco-agent-skills.git .agents/skills/coco

# Or copy individual skills
cp -r coco-agent-skills/coco-intel-briefing .agents/skills/
```

### Manual Install (any compatible tool)

Copy the skill folder to your project's `.agents/skills/` directory (or `.<tool>/skills/` for tool-specific installation).

## What Are Agent Skills?

Agent Skills are an open standard for giving AI coding assistants specialized capabilities. A skill is a folder with a `SKILL.md` file that teaches the agent how to perform a specific task.

When you install a skill, your AI assistant can:
- Recognize when a task matches the skill's domain
- Load the skill's instructions and methodology
- Execute the workflow with your specific context

Learn more at [agentskills.io](https://agentskills.io).

## About COCO

COCO builds AI-powered tools for humans who build and create. Our products include prompt packs, interactive dashboards, and API services.

- Website: [goodstories.world](https://goodstories.world)
- Products: [goodstories.gumroad.com](https://goodstories.gumroad.com)

## License

Apache 2.0 — see [LICENSE](LICENSE) for details.
