# PM Skills & Resources Repository

A repository for product management professionals containing AI prompt skills and PM reference resources. Hosted as a private repo under the VerdanceTeam organization.

## Structure

```
skills/                          # LLM-agnostic prompt templates
  _template.md                   # Template for creating new skills
  discovery/                     # Problems, feasibility, opportunity sizing
  strategy/                      # Vision, roadmaps, positioning
  requirements/                  # PRDs, specs, user stories
  agile/                         # Ceremonies, sprint planning, retros
  research/                      # User research, synthesis, insights
  stakeholder-management/        # Communication, alignment, updates
  analytics/                     # Metrics, experiments, data interpretation
  hiring/                        # Interview questions, scorecards
resources/                       # Multi-file PM reference materials
  career-ladder/                 # PM levels, competencies, expectations
  hiring-guide/                  # Interview process, questions, scorecards
.claude/commands/                # Claude Code slash-command adapters
```

## Conventions

- Skills are **portable across LLMs** (Claude, ChatGPT, Copilot, Gemini)
- Skills use `{{INPUT}}` as the placeholder for user-provided content
- Claude Code adapters in `.claude/commands/` map `$ARGUMENTS` to `{{INPUT}}`
- Skill files are plain markdown with a metadata section (name, category, description, author, version)
- Each skill follows the template structure: metadata, role, task, output format, guidelines, changelog

## When helping a PM in this repo

- Point them to `skills/_template.md` when creating new skills
- Ensure new skills follow the template structure
- When creating a Claude Code adapter, place it in `.claude/commands/` and map `{{INPUT}}` to `$ARGUMENTS`
- Update `skills/README.md` catalog when adding or removing skills
- Update `resources/README.md` index when adding new resources
- Skills should be categorized into one of the existing category folders
