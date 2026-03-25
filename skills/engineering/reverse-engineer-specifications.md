# Reverse Engineer Specifications

Generate accurate Gherkin behavioral specifications by deriving them from source code — routes, components, role checks, status machines, and data files — rather than from memory or product docs. Produces living documentation that reflects what the system actually does, organized around how users navigate it.

---

## Metadata

- **Name**: Reverse Engineer Specifications
- **Category**: engineering
- **Description**: Generates Gherkin `.feature` files from a codebase by tracing routes, reading role checks, extracting status flows, and mapping behavior to the actual navigation structure — not from memory or product docs
- **Author**: VerdanceTeam
- **Version**: 1.0

---

## Prompt

### Role

You are a behavioral specification engineer. Your job is to produce Gherkin feature files that accurately describe what the application does, derived entirely from source code. You trust code, not memory. Every scenario must be traceable to a specific file.

### Task

The following context has been provided:

{{INPUT}}

Work in 3 phases:

---

**Phase 1 — Extract Structure from Code**

Before writing a single scenario, read the codebase to build a complete behavioral map. Run these as parallel agents:

**Agent 1: Navigation & Routes**
- Read the app shell (App.tsx or equivalent) — extract every route definition: path, component, any guards
- Read the sidebar/navigation component — extract every nav item: label, route, visibility conditions
- Map: `{ navLabel → route → component → roleRequired }`
- Note which routes are role-gated at the route level vs component level

**Agent 2: Role & Permission Model**
- Find the role TypeScript type — copy it verbatim (every possible value including edge cases like 'NONE')
- Read the auth component — how is `userRole` determined and stored?
- Read every component that branches on role — extract the exact conditions: `role === 'CMS'`, `['CMS', 'MAC'].includes(role)`, etc.
- Read the environment service or config — which roles can access which environments?
- Produce a complete role × feature permission matrix

**Agent 3: Data & Enumerations**
- Read all data files (JSON, constants, enums) that define domain objects — fee schedules, status values, type enumerations, categories
- For each: extract every item and its properties (key, type, flags, required options)
- Read the status state machine — what are the valid status transitions? (e.g., STAGED → PREVIEWED → APPLIED → PROMOTED)
- Copy enumerations verbatim; never infer from domain knowledge

**Agent 4: Component Behavior**
- For each page component: read the file fully
  - What does it render given different role values?
  - What API calls does it make (or which child components do)?
  - What user actions are available (buttons, forms, table interactions)?
  - What conditional logic determines what the user sees?
- For async operations: how does the UI poll or update state?
- For form submissions: what validations and required fields exist?

**Agent 5: Hook-to-Action Mapping**
- For every mutation hook: what user action triggers it? What are the expected outcomes (success state, error state, status change)?
- For every query hook: what data does it load? What does the UI show when loading / empty / populated?
- Map: `{ userAction → hook → outcomeStates }`

---

**Phase 2 — Write Gherkin Specifications**

Using ONLY the Phase 1 data, write one `.feature` file per navigation section. Apply these rules:

**Structure rules:**
- Mirror the navigation: one directory per nav section, one `.feature` file per major workflow
- Use `Feature:` for the page/section name
- Use `Rule:` to group scenarios by role or condition
- Use `Background:` for shared preconditions (authentication, navigation)
- Use `Scenario:` for each distinct user flow
- Use `Scenario Outline:` with `Examples:` only when the same flow applies to multiple roles or data variants

**Writing rules:**

1. **Role values must match code** — use the exact string values from the TypeScript role type (`'CMS'`, `'MAC'`, `'STE'`) not human-readable labels
2. **Status values must match code** — use exact status strings (`'STAGED'`, `'IN_PROGRESS'`, `'APPLIED'`) not paraphrases
3. **Fee schedule keys must match data** — use exact keys from the data file (`'mpfs'`, `'clinicalLab'`) not display names
4. **Counts must match data** — if the spec mentions "25 fee schedules", count them from the data file
5. **Conditions must match code** — if a button is shown when `status === 'STAGED' || status === 'PREVIEWED'`, the scenario precondition must reflect both
6. **No aspirational scenarios** — only write scenarios for flows that exist in the current code; mark TODOs explicitly if needed
7. **Organize by navigation** — directory structure mirrors the sidebar so specs are findable by anyone who knows the UI
8. **One scenario per distinct outcome** — don't combine multiple outcomes into one scenario; split by success path, error path, and edge cases

**File structure:**
```
docs/specifications/
├── README.md                    # Navigation map, role matrix, status flows, data reference
├── master-workflow.feature      # End-to-end workflow across pages
├── shared/                      # Cross-cutting: auth, header, role switching
├── [nav-section]/               # One dir per nav item
│   └── [workflow].feature       # One file per major workflow
```

**README.md must include:**
- Navigation map (nav label → route → spec file)
- Role × feature permission matrix (derived from Agent 2)
- Status flow diagram (derived from Agent 3)
- Key data reference tables (fee schedule types, enumeration values, etc.)

---

**Phase 3 — Self-Validate**

Before presenting, run these checks:

1. **Role value check** — every role string in every scenario matches the TypeScript role type exactly
2. **Status value check** — every status string matches the actual status enumeration exactly
3. **Data key check** — every fee schedule key, type name, or enumeration value exists in the data files
4. **Coverage check** — every route in the navigation map has at least one feature file; flag any uncovered routes
5. **Condition accuracy check** — for each scenario precondition involving a status or role, verify it matches the actual conditional logic in the component

Fix any failures before presenting.

---

**Present with:**
- Summary: how many feature files, total scenarios, coverage (routes documented vs total routes)
- Any routes or flows intentionally omitted and why
- Self-validation results
- Ask: write to `docs/specifications/`? Any sections to expand?

### Output Format

One `.feature` file per workflow, organized in directories mirroring the application navigation. A `README.md` with navigation map, permission matrix, status flows, and data reference tables.

### Guidelines

- Read role checks from code — don't assume which roles can do what
- Read status transitions from code — don't assume a conventional CRUD flow
- Read data files for enumerations — don't invent plausible values
- Treat absence of a feature flag or route guard as evidence that access is unrestricted
- Scenarios describe observable user behavior, not implementation details — no RTK Query hook names in Gherkin steps
- When the same behavior applies to multiple roles, use `Scenario Outline` with an `Examples` table rather than duplicating scenarios
- Living documentation: specs derived from code stay accurate as long as the derivation process is repeated when the code changes

---

## Claude Code Setup

To make this skill available as a slash command in Claude Code, create a file at `.claude/commands/reverse-engineer-specifications.md`:

```markdown
---
description: Generate Gherkin behavioral specifications from source code
---

Generate Gherkin behavioral specifications by reverse engineering the actual source code.

[paste the Phase 1–3 Task prompt above here, replacing {{INPUT}} with $ARGUMENTS]
```

Then invoke with: `/reverse-engineer-specifications [optional: scope or directory]`

---

## Changelog

| Version | Date | Changes |
|---|---|---|
| 1.0 | 2026-03-15 | Initial version, derived from PCS-UI BDD specification generation (16 feature files, 116 scenarios) |
