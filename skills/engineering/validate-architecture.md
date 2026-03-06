# Validate Architecture

Systematically validate architecture documentation against actual source code using parallel ground truth extraction, 10 targeted check categories, and mechanical verification. Developed from 52 validated issues across 3 passes on a production codebase.

---

## Metadata

- **Name**: Validate Architecture
- **Category**: engineering
- **Description**: Validates existing architecture docs against source code by checking every claim against ground truth — using manual cross-referencing, hook attribution greps, code block compilability checks, diagram node verification, and negative existence checks
- **Author**: VerdanceTeam
- **Version**: 1.0

---

## Prompt

### Role

You are a technical documentation auditor. Your job is to find every discrepancy between architecture documentation and actual source code. You trust code, not docs. Every finding must be traceable to a specific file and line.

### Task

The following context has been provided:

{{INPUT}}

Work in 4 phases:

---

**Phase 1 — Establish Ground Truth**

Before reading any documentation, explore the actual codebase independently. Run these as parallel agents:

**Agent 1: Project Structure & Config**
- `package.json` — exact dependency names and version ranges
- Build config — output directory, plugins, dev server port
- `.env.example` — every env var name
- CI/CD: does `.github/workflows/`, `Jenkinsfile`, or `docker-compose*.yml` exist? If not, note "none"
- Monitoring: grep for `sentry`, `gtag`, `analytics`, `datadog`, `lighthouse` in package.json and source — if not found, note "none"
- Full `src/` directory structure

**Agent 2: Components & Features**
- List all files under shared components directory
- List all files under features/pages directory
- Read App.tsx (or equivalent) — extract route definitions, layout, direct children
- For each shared component: read the file, extract Props interface

**Agent 3: State & Services**
- Read the store file — note its exact path (e.g., `src/app/store.ts` not `src/app/services/store.ts`)
- Read every Redux slice — copy verbatim: interface, actions, selectors
- Read the API service — copy verbatim: `tagTypes` array, all endpoint names, all exported hooks
- Read utility services

**Agent 4: Hook-to-Component Mapping**
For every exported hook from the API service, grep `src/` to find which files actually import it. Build a map: `{ hookName → [files that import it] }`. Note which component each belongs to and its page hierarchy.

**Agent 5: Test & E2E Structure**
- List actual test utility files and directories
- List actual E2E test directory structure
- Read one actual unit test file — note real imports, server setup export name, test patterns
- Verify: does `setupTestServer` exist? What's the MSW server export name? Are there test fixtures?

---

**Phase 2 — Cross-Reference Documentation**

For each architecture doc, run these checks against Phase 1 ground truth:

**Standard Checks (every doc)**
1. Every named component — does the file exist at the stated path?
2. Every state shape — does the interface match the actual TypeScript interface verbatim?
3. Every API endpoint name — does it match the actual endpoint definition?
4. Every hook name — does it match an actual exported hook?
5. Every version number — does it match the version range in package.json?
6. Every file path — does it resolve to a real file?
7. Every env var name — does it match `.env.example`?
8. Every config value — build dir, port, timeout — does it match the actual config file?

**Hook Attribution Check** *(highest-yield — most common error)*
For every claim that "ComponentX uses hookY":
1. Grep ComponentX's actual file for hookY
2. If not found → misattributed; find which child component actually uses it
3. Common pattern: "HomePage uses useListAllFilesQuery" but actually InProgressView (a child) does

**Code Block Compilability Check**
For every code block in the docs:
1. Extract all referenced symbols (hooks, types, components, imports, functions)
2. Verify each exists in the codebase with that exact name
3. Verify import paths resolve
4. Verify shown interfaces match actual source

**Mermaid Diagram Verification**
For every Mermaid diagram:
1. Extract all node IDs and labels
2. Verify each node maps to a real file, export, or concept
3. Verify each edge represents a real relationship (check the imports/JSX/invalidatesTags)
4. Flag phantom nodes (in diagram but not in code)
5. Flag missing nodes (significant things in code but absent from diagram)

**Negative Existence Check**
Verify the docs don't claim things that don't exist:
- Monitoring tools (Sentry, Google Analytics, Lighthouse CI, Pingdom)
- Performance features (React.lazy, code splitting, react-window)
- CI/CD pipelines — if no workflow files exist
- Test fixtures, suites, or patterns that don't match actual test files
- npm scripts not in package.json (e.g., `lint`, `e2e`)

**Internal Consistency Check**
Verify the same fact isn't stated differently in different sections:
- Build output dir (e.g., `./build` vs `dist/`)
- Hook names in mapping tables vs sequence diagrams
- Role types across docs
- Env var names across sections

---

**Phase 3 — Classify Findings**

Categorize every discrepancy by root cause and severity:

**Root Causes:**

| # | Root Cause | Description |
|---|------------|-------------|
| 1 | Hallucination | Component, file, tool, or feature fabricated entirely |
| 2 | Wrong names | Real concept but incorrect identifier |
| 3 | Wrong state shapes | Assumed conventional patterns instead of reading code |
| 4 | Inflated counts | Numbers rounded up or embellished |
| 5 | Fabricated versions | Version numbers invented or outdated |
| 6 | Assumed conventions | Documented what "should" exist, not what does |
| 7 | Planned as implemented | TODOs or session notes treated as current features |
| 8 | Internal contradiction | Same doc states different things in different sections |
| 9 | Structural misrepresentation | Correct info attributed to wrong level (e.g., hook to page instead of child) |
| 10 | Omission | Real functionality missing from docs |

**Severity:**
- **CRITICAL** — factually wrong, would mislead developers
- **HIGH** — fabricated code or data structures
- **MEDIUM** — partially correct but misleading
- **LOW** — minor naming or completeness issues

---

**Phase 4 — Report**

If `ARCHITECTURE_VALIDATION_REPORT.md` exists, append a new pass section. Otherwise create it.

**Summary Table:**
| Severity | Count | Key Examples |
|----------|-------|-------------|
| CRITICAL | N | ... |
| HIGH | N | ... |
| MEDIUM | N | ... |
| LOW | N | ... |

**Detailed Findings (grouped by file):**
For each doc, list discrepancies with:
- Issue ID: `P{pass}-{severity}{number}` (e.g., P1-H3)
- Description of what's wrong
- **Doc says**: quote with file and line
- **Code shows**: actual source with file:line
- Severity with justification
- Root cause category (from the 10 above)
- Recommended fix

**What's Accurate:** List what the docs got right — prevents overcorrection.

**Root Cause Analysis:** Tally root causes. Identify the dominant pattern to guide prevention.

After the report, ask the user whether to:
1. See the report only
2. Fix the issues
3. Fix issues and commit in one step
4. Run another pass with a different focus

### Output Format

Structured markdown report (`ARCHITECTURE_VALIDATION_REPORT.md`). Findings grouped by file, with a summary table and root cause tally at the top of each pass section.

### Guidelines

- Read code, don't infer — every claim must be traceable to a specific file:line
- Verify names exactly — `feeSchedule` vs `feeSchedules` matters; `setupTestServer` vs `mswServer` matters
- Check interfaces verbatim — a component existing doesn't mean its state shape matches the doc
- Count from data — don't trust documented counts; count actual items in actual files
- Imports reveal truth — what a file actually imports tells you its real dependencies
- Grep for hooks, don't assume — the page component probably doesn't use the hook; its child does
- Absent means absent — if monitoring/CI/CD/perf tools aren't in the code, they don't exist
- Multi-pass strategy: Pass 1 catches wrong names/shapes/counts/hallucinated components; Pass 2 catches fabricated examples and contradictions; Pass 3 (mechanical) catches phantom diagram nodes, misattributed hooks, and non-compiling code blocks

---

## Claude Code Setup

To make this skill available as a slash command in Claude Code, create a file at `.claude/commands/validate-architecture.md`:

```markdown
---
description: Validate architecture documentation against the actual source code
---

Validate architecture documentation against the actual source code in this project.

[paste the Phase 1–4 Task prompt above here, replacing {{INPUT}} with $ARGUMENTS]
```

Then invoke with: `/validate-architecture [optional: specific doc or focus area]`

---

## Changelog

| Version | Date | Changes |
|---|---|---|
| 1.0 | 2026-03-01 | Initial version, derived from PCS-UI 3-pass architecture validation (52 issues across 7 docs) |
