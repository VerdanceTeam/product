# PM Skills & Resources Repository

A repository for product management professionals containing AI prompt skills and PM reference resources. Hosted as a private repo under the VerdanceTeam organization.

## Structure

```
skills/                          # LLM-agnostic prompt templates
  _template.md                   # Template for creating new skills
  discovery/                     # Problems, feasibility, opportunity sizing
  strategy/                      # Vision, roadmaps, positioning
  product-definition/            # PRDs, specs, user stories
  tactical-execution/            # Sprint ceremonies, planning, retros, shipping
  service-design/                # User research, synthesis, insights, journey mapping
  stakeholder-management/        # Communication, alignment, updates
  measurement/                   # Metrics, validation, experimentation, product effectiveness
  hiring/                        # Interview questions, scorecards
  meta/                          # Using this repo, AI growth, prompt engineering
    eureka.md                    # Capture in-the-moment lesson → skill or GitHub issue
    elementary.md                # Scan context → reflections file → promote to Eureka
  uncategorized/                 # Skills not yet assigned to a category
resources/                       # Multi-file PM reference materials
  career-ladder/                 # PM levels, competencies, expectations
  hiring-guide/                  # Interview process, questions, scorecards
reflections/                     # Output files from /elementary sessions (YYYY-MM-DD-elementary.md)
.claude/commands/                # Claude Code slash-command adapters
  eureka.md                      # /eureka — in-the-moment lesson capture
  elementary.md                  # /elementary — context scan with file output and promotion flow
```

## Conventions

- Skills are **portable across LLMs** (Claude, ChatGPT, Copilot, Gemini)
- Skills use `{{INPUT}}` as the placeholder for user-provided content
- Claude Code adapters in `.claude/commands/` map `$ARGUMENTS` to `{{INPUT}}`
- Skill files are plain markdown with a metadata section (name, category, description, author, version)
- Each skill follows the template structure: metadata, role, task, output format, guidelines, changelog
- Categories are a first cut and will be revisited as skills are added and our PM practice evolves

## Git Workflow

- The initial commit may be pushed directly to main. All subsequent changes go through pull requests.
- PRs must include descriptive context (summary of what changed and why).
- When pushing additional commits to a PR branch that change its scope, update the PR description to reflect what will actually be merged. If the description cannot be fully rewritten, add a comment summarizing what changed and why.
- When merging a PR, delete the source branch.
- After merging, update this file (CLAUDE.md) with any new context from the PR.

## Meta Skills

Two skills exist for capturing and developing PM/AI learnings:

- **`/eureka [lesson]`** — in-the-moment capture. Takes an explicit insight, confirms it, then decides skill vs GitHub issue and drafts accordingly.
- **`/elementary [context]`** — deliberate review. Scans pasted context or the current conversation, writes all candidate lessons to `reflections/YYYY-MM-DD-elementary.md`, asks the user which to promote, updates the file (EUREKA / Passed), then cascades promoted lessons into the eureka flow.

These two skills cascade: elementary delegates the skill/issue decision and drafting to the eureka skill (`skills/meta/eureka.md` Steps 3–4).

## When helping a PM in this repo

- Point them to `skills/_template.md` when creating new skills
- Ensure new skills follow the template structure
- When creating a Claude Code adapter, place it in `.claude/commands/` and map `{{INPUT}}` to `$ARGUMENTS`
- Update `skills/README.md` catalog when adding or removing skills
- Update `resources/README.md` index when adding new resources
- Skills should be categorized into one of the existing category folders
