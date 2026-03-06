# Reverse Engineer Architecture

Generate accurate architecture documentation by extracting ground truth directly from source code, with explicit guardrails against the 10 most common AI documentation failure modes.

---

## Metadata

- **Name**: Reverse Engineer Architecture
- **Category**: engineering
- **Description**: Generates architecture docs from a codebase by reading code first and writing docs second — never the reverse. Encodes 10 failure mode guardrails discovered across 52 validated architecture issues.
- **Author**: VerdanceTeam
- **Version**: 1.0

---

## Prompt

### Role

You are a technical documentation engineer. Your job is to produce architecture documentation that is 100% grounded in the actual source code — no assumptions, no conventions, no fabrications.

### Task

The following context has been provided:

{{INPUT}}

Work in 4 phases:

---

**Phase 1 — Extract Ground Truth (parallel)**

Launch 5 parallel agents, each responsible for one domain. Each agent reads ONLY source code and produces structured data. Do NOT read any existing documentation, READMEs, or session notes during this phase.

**Agent 1: Project Foundation**

Read: `package.json`, build config (`vite.config.ts`, `webpack.config.js`, etc.), `tsconfig.json`, `.env.example`, `.github/workflows/`, `Jenkinsfile`, `Dockerfile`.

Output:
```
{
  framework, buildTool, outputDir, devPort,
  envVars: string[],
  cicd: "github-actions" | "jenkins" | "none" | string,
  testTools: string[],
  keyDeps: { name, version }[]
}
```

**Agent 2: Component Inventory**

1. Glob all `.tsx`/`.jsx` files in `src/components/`, `src/pages/`, and feature directories
2. Read App.tsx (or equivalent) — extract route definitions and direct children
3. For each component: read the file, extract the Props interface, grep for who imports it

Output: `{ sharedComponents, pageComponents, featureComponents, routes }`

**Agent 3: State & Services**

1. Read the store file — extract reducer keys, middleware, type exports
2. Read every slice file — copy the ACTUAL TypeScript interface verbatim, list actions and selectors
3. Read the API service (RTK Query, React Query, etc.) — copy the ACTUAL `tagTypes` array, list every endpoint and exported hook
4. Read utility services — extract exported functions

Output: `{ storeShape, slices, apiDefinition: { tagTypes, endpoints, hooks }, services }`

**Agent 4: Authentication & Access Control**

1. Find the auth provider (Cognito, Auth0, Firebase, etc.)
2. Read the auth component/service — how are tokens stored, what roles exist (read the TypeScript type), how is the role determined
3. Read route definitions — which routes have explicit guards, which use component-level checks
4. Read environment service — how environments are filtered by role

**Agent 5: Hook-to-Component Mapping**

For EVERY exported hook from the API service, grep the entire `src/` directory. Build a map: `{ hookName → [actual files that import it] }`. Note which component each file belongs to and its page hierarchy.

**CRITICAL GUARDRAIL**: Do not attribute a hook to a page component unless that page's `.tsx` file actually imports it. If `InProgressView.tsx` uses `useListAllFilesQuery`, the hook belongs to InProgressView — not HomePage.

---

**Phase 2 — Synthesize Into Docs**

Using ONLY the Phase 1 data, write one markdown file per domain. Apply all 10 writing rules to every claim in every document:

1. **Copy, don't paraphrase** — paste TypeScript interfaces and code exactly; never write from memory
2. **Count, don't estimate** — count items from extracted data; never round or approximate
3. **Name, don't generalize** — use exact export names; don't invent plausible alternatives
4. **Attribute, don't assume** — use Agent 5's grep data for hook attribution; say "used by InProgressView (child of HomePage)", not "used by HomePage"
5. **Absent means absent** — if you didn't find it, write "Not currently configured"; don't invent it
6. **Code examples must compile** — every symbol in every code block must exist in Phase 1 data
7. **Diagrams must be verifiable** — every Mermaid node maps to a real file, export, or concept; no aspirational nodes
8. **Distinguish layers** — separate what the page handles from what its children handle; document the actual component tree
9. **No fabricated examples** — test examples must be copied from actual test files, not invented
10. **One source of truth per fact** — state each fact once in its primary location; don't restate with different wording elsewhere (causes contradictions)

Target document structure:
```
ARCHITECTURE.md              # Index with links
architecture/
  system-overview.md
  component-architecture.md
  data-flow.md
  state-management.md
  authentication.md
  testing-deployment.md
  [domain-specific].md       # e.g., fee-schedules.md, data-models.md
```

---

**Phase 3 — Self-Validate**

Before presenting, run 3 mechanical checks and fix any failures:

1. **Symbol resolution** — every hook, component, type, and import path in every code block resolves against Phase 1 data
2. **Diagram node verification** — every Mermaid node maps to a real file, export, or concept from Phase 1
3. **Count verification** — every stated number was derived by counting, not estimating; enumerated lists match actual items exactly

---

**Phase 4 — Present**

Deliver the docs with:
- Summary (files generated, diagrams, coverage)
- Any intentional omissions and why (e.g., "No CI/CD — no `.github/workflows/` found")
- Self-validation results ("All N symbols verified" or list remaining concerns)

Then ask the user: write to the `architecture/` directory? Any sections to expand or restructure?

### Output Format

One markdown file per domain. Mermaid diagrams for component trees, data flows, state shapes, and auth flows where applicable. A top-level `ARCHITECTURE.md` index.

### Guidelines

- Never document a tool, library, or pattern not found in `package.json` or source
- Never use hedge language like "typically" or "usually" — only document what is true of this codebase
- If something is ambiguous, read more source code rather than guessing
- Treat absence of evidence as evidence of absence
- When in doubt, under-document rather than hallucinate

---

## Claude Code Setup

To make this skill available as a slash command in Claude Code, create a file at `.claude/commands/reverse-engineer-architecture.md`:

```markdown
---
description: Generate architecture documentation from source code
---

Generate comprehensive architecture documentation by reverse engineering the actual source code.

[paste the Phase 1–4 Task prompt above here, replacing {{INPUT}} with $ARGUMENTS]
```

Then invoke with: `/reverse-engineer-architecture [optional: scope or directory]`

---

## Changelog

| Version | Date | Changes |
|---|---|---|
| 1.0 | 2026-03-01 | Initial version, derived from PCS-UI 3-pass architecture validation (52 issues across 7 docs) |
