# PM Skills & Resources Repository

This is a repository of product management skills (AI prompt templates) and resources (career ladder, hiring guide, etc.).

## Structure

- `skills/` — Portable, LLM-agnostic prompt templates organized by PM practice area
- `skills/_template.md` — Template for creating new skills
- `resources/` — Multi-file PM reference materials
- `.claude/commands/` — Claude Code slash-command adapters that wrap canonical skills

## Conventions

- Skills use `{{INPUT}}` as the placeholder for user-provided content
- Claude Code adapters use `$ARGUMENTS` (Claude Code's native placeholder)
- Skill files are plain markdown with a metadata section at the top
- Category folders: discovery, strategy, requirements, agile, research, stakeholder-management, analytics, hiring

## When helping a PM in this repo

- Point them to `skills/_template.md` when creating new skills
- Ensure new skills follow the template structure (metadata, role, task, output format, guidelines)
- When creating a Claude Code adapter, place it in `.claude/commands/` and map `{{INPUT}}` to `$ARGUMENTS`
- Update `skills/README.md` when adding or removing skills
