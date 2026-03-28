# COCO Agent Skills

> Tools for humans building and creating with AI

16 open-source [Agent Skills](https://agentskills.io) for business operations, engineering, content creation, and data analysis. Compatible with **32+ AI coding tools** including Claude Code, Cursor, VS Code Copilot, Gemini CLI, OpenAI Codex, and more.

## Quick Install

```bash
# Install all skills (any agent that supports npx skills)
npx skills add goodstoriesworld/coco-agent-skills

# Or clone directly
git clone https://github.com/goodstoriesworld/coco-agent-skills.git .agents/skills/coco
```

## COCO Originals

Skills built by COCO — unique workflows you won't find elsewhere.

### Business & Operations

| Skill | Description |
|-------|-------------|
| [coco-intel-briefing](coco-intel-briefing/) | Build a daily intelligence briefing pipeline from RSS feeds |
| [coco-content-creator](coco-content-creator/) | Generate multi-platform content (LinkedIn, Twitter/X, email, blog) from a single input |
| [coco-business-ops](coco-business-ops/) | Business operations assistant — revenue, expenses, clients, content, goals |
| [coco-marketing-planner](coco-marketing-planner/) | Generate complete marketing campaigns from product details |
| [coco-budget-planner](coco-budget-planner/) | Build budgeting tools with savings projections and allocation strategies |

### Engineering & Data

| Skill | Description |
|-------|-------------|
| [coco-multi-agent-system](coco-multi-agent-system/) | Build production multi-agent systems that plan, implement, test, and evaluate autonomously |
| [coco-data-pipeline](coco-data-pipeline/) | Build CSV analytics pipelines with validation, insight generation, and visualization |
| [coco-sustainability-analyzer](coco-sustainability-analyzer/) | Evaluate and optimize sustainability impact for engineering projects |

## Curated Collection

Top open-source skills from the community, vendored with proper attribution. All Apache 2.0 licensed.

### Developer Tools

| Skill | Description | Original Source |
|-------|-------------|-----------------|
| [code-reviewer](vendored/code-reviewer/) | Thorough code reviews with structured feedback | TerminalSkills |
| [test-generator](vendored/test-generator/) | Auto-generate comprehensive test suites | TerminalSkills |
| [git-commit-pro](vendored/git-commit-pro/) | Write clear conventional commit messages | TerminalSkills |
| [security-audit](vendored/security-audit/) | Audit code for security vulnerabilities | TerminalSkills |
| [sql-optimizer](vendored/sql-optimizer/) | Optimize SQL queries for performance | TerminalSkills |
| [frontend-design](vendored/frontend-design/) | Build production-grade frontend interfaces | Anthropic |
| [api-designer](vendored/api-designer/) | Design clean REST and GraphQL APIs | COCO |
| [prompt-engineering](vendored/prompt-engineering/) | Write effective prompts using proven patterns | ok-skills |

See [vendored/NOTICE.md](vendored/NOTICE.md) for full attribution.

## Compatible Tools

Works with any tool that supports the [Agent Skills](https://agentskills.io) open standard:

Claude Code, Cursor, VS Code Copilot, GitHub Copilot, OpenAI Codex, Gemini CLI, JetBrains Junie, Kiro, Roo Code, Goose, Amp, OpenHands, OpenCode, and 20+ more.

## Install Individual Skills

```bash
# Install just one skill
npx skills add goodstoriesworld/coco-agent-skills --skill coco-intel-briefing

# Or copy manually
cp -r coco-agent-skills/coco-intel-briefing .agents/skills/
```

## About COCO

COCÓ builds AI-powered tools for humans who build and create. Products include interactive dashboards, API services, prompt packs, and agent skills.

- Website: [goodstories.world](https://goodstories.world)
- Products: [goodstories.gumroad.com](https://goodstories.gumroad.com)

## License

Apache 2.0 — see [LICENSE](LICENSE) for details.
Vendored skills retain their original licenses — see individual SKILL.md files for details.
