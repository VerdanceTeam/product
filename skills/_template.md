# Skill Template

Use this template to create a new PM skill. Copy this file into the appropriate category folder and rename it (e.g., `skills/requirements/prd.md`).

---

## Metadata

<!-- Update these fields for your skill -->

- **Name**: Skill Name
- **Category**: category-name
- **Description**: One-line description of what this skill does
- **Author**: Your Name
- **Version**: 1.0

---

## Prompt

<!--
  This is the core prompt that gets sent to the LLM.

  Guidelines:
  - Start with a role/persona to set context
  - Use {{INPUT}} as the placeholder for user-provided content
  - Be specific about the output format you want
  - Include constraints and guidelines to shape quality
  - Keep it self-contained — the LLM won't have other context
-->

### Role

You are an experienced product manager with deep expertise in [relevant domain].

### Task

[Describe what the LLM should do. Be specific about the deliverable.]

The following context has been provided:

{{INPUT}}

### Output Format

<!-- Define the structure of the expected output -->

Provide your response in the following format:

1. **Section Name**: Description of what goes here
2. **Section Name**: Description of what goes here
3. **Section Name**: Description of what goes here

### Guidelines

<!-- Rules, constraints, and quality criteria -->

- Guideline 1
- Guideline 2
- Guideline 3

---

## Claude Code Setup

To make this skill available as a slash-command in Claude Code, create a file at `.claude/commands/<skill-name>.md` with the following content:

```markdown
---
description: One-line description (same as metadata above)
---

<paste the Prompt section above here, replacing {{INPUT}} with $ARGUMENTS>
```

Then you can invoke it with: `/skill-name your input here`

---

## Changelog

| Version | Date | Changes |
|---|---|---|
| 1.0 | YYYY-MM-DD | Initial version |
