# PM Skills & Resources

A repository of skills, frameworks, and resources for product management professionals. Skills are portable AI prompt templates that work with any LLM — Claude, ChatGPT, Copilot, Gemini, and others.

## Quick Start

1. Clone this repo
2. Browse the [Skill Catalog](skills/README.md) to see available skills
3. Use skills with your preferred AI assistant (see below)
4. Create your own skills using the [template](skills/_template.md)

## Repository Structure

```
skills/           AI prompt templates organized by PM practice area
resources/        Reference materials (career ladder, hiring guide, etc.)
.claude/commands/ Claude Code slash-command adapters
```

## How to Use Skills

### With Claude Code

Skills are available as slash-commands. Open Claude Code in this project and type:

```
/skill-name your input here
```

Claude Code reads the skill prompt and applies it to your input. See [.claude/commands/README.md](.claude/commands/README.md) for setup details.

### With Other LLMs (ChatGPT, Copilot, Gemini, etc.)

1. Open the skill file from `skills/<category>/`
2. Copy the prompt content
3. Paste it into your LLM of choice
4. Replace `{{INPUT}}` with your specific context

## Skill Categories

> **Note:** These categories are a first cut and will be revisited as skills are added and our PM practice evolves. Expect categories to be renamed, merged, or restructured over time.

| Category | Description |
|---|---|
| [discovery/](skills/discovery/) | Understanding problems, technical feasibility, opportunity sizing |
| [strategy/](skills/strategy/) | Vision, roadmaps, positioning, competitive analysis |
| [product-definition/](skills/product-definition/) | PRDs, specs, user stories, acceptance criteria |
| [tactical-execution/](skills/tactical-execution/) | Sprint ceremonies, planning, retros, shipping |
| [service-design/](skills/service-design/) | User research, synthesis, insights, journey mapping |
| [stakeholder-management/](skills/stakeholder-management/) | Communication, alignment, status updates |
| [measurement/](skills/measurement/) | Metrics, validation, experimentation, measuring product effectiveness |
| [hiring/](skills/hiring/) | Interview questions, scorecards |
| [meta/](skills/meta/) | Using this repo effectively, AI growth, prompt engineering |
| [uncategorized/](skills/uncategorized/) | Skills not yet assigned to a category |

See the full [Skill Catalog](skills/README.md) for individual skill listings.

## Resources

Reference materials for PM practice:

| Resource | Description |
|---|---|
| [Career Ladder](resources/career-ladder/) | PM levels, competencies, and expectations |
| [Hiring Guide](resources/hiring-guide/) | Interview process, questions, and scorecards |

See the [Resources Index](resources/README.md) for details.

## Creating a New Skill

1. Copy `skills/_template.md` into the appropriate category folder
2. Rename it to your skill name (e.g., `prd.md`)
3. Fill in the metadata, prompt, and guidelines
4. Add an entry to the [Skill Catalog](skills/README.md)
5. Optionally, create a Claude Code adapter in `.claude/commands/`

See the [template](skills/_template.md) for detailed instructions and best practices.
