# Claude Code Slash-Command Adapters

This folder contains Claude Code slash-command files that wrap the canonical skills from `skills/`.

## How It Works

Skills in this repo are written as LLM-agnostic markdown prompts using `{{INPUT}}` as a placeholder. To use them as Claude Code slash-commands, you create a thin adapter file here that:

1. Uses Claude Code's frontmatter format (`description` field)
2. Contains the skill prompt with `$ARGUMENTS` instead of `{{INPUT}}`

## Creating an Adapter

1. Start with a canonical skill from `skills/<category>/skill-name.md`
2. Create a new file here: `.claude/commands/skill-name.md`
3. Add the frontmatter and copy the prompt section:

```markdown
---
description: Brief description of the skill
---

<Copy the prompt from the canonical skill file>

<Replace {{INPUT}} with $ARGUMENTS>
```

## Example

Given a canonical skill at `skills/requirements/prd.md`, the adapter at `.claude/commands/prd.md` would look like:

```markdown
---
description: Generate a Product Requirements Document
---

You are an experienced product manager with deep expertise in product development.

Create a comprehensive PRD for the following:

$ARGUMENTS

Provide your response in the following format:

1. **Problem Statement**: What problem are we solving and for whom
2. **Goals & Success Metrics**: Measurable outcomes
3. **User Stories**: Key user stories with acceptance criteria
4. **Requirements**: Functional and non-functional requirements
5. **Open Questions**: Assumptions and unknowns to resolve
```

A PM would then use it by typing: `/prd AI-powered search feature for our SaaS platform`

## Naming Convention

- File name becomes the command name: `prd.md` -> `/prd`
- Use lowercase kebab-case: `user-story.md` -> `/user-story`
- Keep names short and memorable
