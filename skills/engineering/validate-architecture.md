# Validate Architecture

Systematically validate architecture documentation against actual source code using parallel ground truth extraction, targeted check categories, and mechanical verification. Developed from 52 validated issues across 3 passes on a production codebase, then field-tested on a 4-repo Go/React system (MDPP).

---

## Metadata

- **Name**: Validate Architecture
- **Category**: engineering
- **Description**: Validates existing architecture docs against source code by checking every claim against ground truth — using repo discovery, stack detection, manual cross-referencing, hook attribution greps, code block compilability checks, Mermaid CLI rendering, diagram node verification, inter-repo data contract checks, and negative existence checks
- **Author**: VerdanceTeam
- **Version**: 1.1

---

## Prompt

### Role

You are a technical documentation auditor. Your job is to find every discrepancy between architecture documentation and actual source code. You trust code, not docs. Every finding must be traceable to a specific file and line.

### Task

The following context has been provided:

{{INPUT}}

Work in 4 phases:

---

**Phase 0 — Discover Repos & Detect Stacks**

Before exploring any source code, read the architecture docs to find out which repos are being documented. This prevents hardcoding assumptions about project structure.

1. Read every `.md` file in the architecture docs directory
2. Extract every repo name, component, or source directory referenced
3. Locate each on disk — check sibling directories (`../`, `../../`, etc.)
4. Build a map: `{ docFile → [source repo paths] }`
5. Note any referenced repos that cannot be found — these are automatic findings (severity: HIGH)

For each found repo, detect the stack:
- `package.json` → Node/JS — check `dependencies` for React, Express, Next.js, etc.
- `pom.xml` / `build.gradle` → Java/JVM
- `requirements.txt` / `pyproject.toml` → Python
- `go.mod` → Go
- `Cargo.toml` → Rust
- `*.csproj` / `*.sln` → .NET
- `Dockerfile` / `docker-compose.yml` → containerized (note regardless of stack)
- `template.yaml` / `serverless.yml` / `cdk.json` → AWS Lambda / IaC
- `*.sql` files at root or `/sql` / `/migrations` → SQL-heavy data pipeline

Unknown or mixed stacks: note explicitly — don't guess.

---

**Phase 1 — Establish Ground Truth**

Using the repo map and detected stacks from Phase 0, explore each repo. Run these in parallel.

**Agent 1: Project Structure & Config (all stacks)**
- `package.json` or language equivalent — exact dependency names and version ranges
- Build config — output directory, plugins, dev server port
- `.env.example` — every env var name and example value
- CI/CD: does `.github/workflows/`, `Jenkinsfile`, or `docker-compose*.yml` exist? If not, note "none"
- Monitoring: grep for `sentry`, `gtag`, `analytics`, `datadog`, `lighthouse`, `newrelic` in config and source — if not found, note "none"
- Full source directory structure

**Agent 2: Components & Features (frontend repos)**
- List all files under shared components directory
- List all files under features/pages directory
- Read App.tsx or equivalent — extract route definitions, layout, direct children
- For each shared component: read the file, extract Props interface

**Agent 3: State & Services (frontend repos)**
- Read the store file — note its exact path
- Read every Redux slice — copy verbatim: interface, actions, selectors
- Read the API service — copy verbatim: `tagTypes`, all endpoint names, all exported hooks
- Read utility services

**Agent 4: Hook-to-Component Mapping (frontend repos)**
For every exported hook from the API service, grep `src/` to find which files actually import it. Build a map: `{ hookName → [files that import it] }`.

**Agent 5: Test & E2E Structure (all stacks)**
- List actual test utility files and directories
- List actual E2E test directory structure
- Read one actual unit test file — note real imports, server setup export name, test patterns
- Verify: does `setupTestServer` exist? What's the MSW server export name? Are there test fixtures?

**Agent 6: REST API routes (Node/Express repos)**
- Extract all route definitions — method, path, handler name
- Check for OpenAPI/Swagger spec — if present, extract all endpoint paths and methods
- List middleware applied globally vs per-route
- Note authentication mechanism (JWT, session, API key, OAuth)

**Agent 7: Python API routes (Python repos)**
- Extract all route decorators (`@app.get`, `@router.post`, etc.) — list method, path, function name
- Check for OpenAPI spec generation
- List dependencies with exact pinned versions

**Agent 8: Java/Spring routes (Java repos)**
- Extract all `@RequestMapping`, `@GetMapping`, `@PostMapping`, etc.
- Check for OpenAPI spec (`springdoc-openapi`, `springfox`)
- Extract DTOs/request-response models

**Agent 9: Data Pipeline / ETL (pipeline repos)**
- List all SQL files and their purpose
- Identify trigger mechanism: cron schedule, event-driven (SQS, S3, SNS), or manual
- Extract input sources and output destinations — note exact names (bucket names, table names, queue names)
- List transformation steps in order with actual function/class names

**Agent 10: AWS Lambda / Serverless (IaC repos)**
- Read `template.yaml`, `serverless.yml`, or `cdk.json` — extract every function name, trigger type, handler path, timeout, memory
- List environment variables defined in IaC config
- Note IAM roles and permissions

---

**Phase 2 — Cross-Reference Documentation**

For each architecture doc, run these checks against Phase 1 ground truth:

**Standard Checks (every doc)**
1. Every named component — does the file exist at the stated path?
2. Every state shape — does the interface match the actual TypeScript/Go/Python interface verbatim?
3. Every API endpoint name — does it match the actual definition?
4. Every hook name — does it match an actual exported hook?
5. Every version number — does it match the version range in the package manifest?
6. Every file path — does it resolve to a real file?
7. Every env var name — does it match `.env.example` or IaC config?
8. Every config value — build dir, port, timeout — does it match the actual config?

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

**Mermaid Diagram Syntax Validation (run the CLI — don't just read the diagram)**
1. Extract each Mermaid block (everything between ` ```mermaid ` and ` ``` `)
2. Write each to a temp file: `/tmp/diagram_N.mmd`
3. Run: `npx @mermaid-js/mermaid-cli -i /tmp/diagram_N.mmd -o /tmp/diagram_N.svg 2>&1`
4. Any non-zero exit or parse error → HIGH finding. Include the exact error message.
5. Clean up temp files after.

Common Mermaid parse errors:
- Reserved words in labels (`end`, `start`, `default`, `class`, `style`) — quote the label: `A["normalize end state"]`
- **Semicolons in sequence diagram labels** — `;` acts as a statement terminator in Mermaid's lexer; replace with `,` or rephrase: `Lambda->>Lambda: Enrich record with Lat, Lon; pad ZipCode` fails, use `,` instead
- Missing arrow between nodes on the same line
- Subgraph not closed with `end` on its own line

**Mermaid Diagram Semantic Verification** *(diagrams that pass syntax check)*
1. Verify each node maps to a real file, export, or concept
2. Verify each edge represents a real relationship (check imports/JSX/invalidatesTags)
3. Flag phantom nodes (in diagram but not in code)
4. Flag missing nodes (significant things in code but absent from diagram)

**Inter-Repo Data Contract Check** *(multi-repo systems only)*
When two repos share a data contract over a queue, cache, or storage layer:
1. Find the writer repo's serialization struct and extract its JSON/field tags
2. Find the reader repo's deserialization struct and extract its JSON/field tags
3. Verify every field name matches exactly — including whitespace. A leading or trailing space in a Go JSON tag (e.g., `json:" Organization Name"`) silently produces empty fields at runtime with no error.
4. Verify shared keys (Redis keys, SQS message fields, S3 object key patterns) are identical in both repos

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

Categorize every discrepancy by **finding type**, root cause, and severity.

**Finding Type — classify every finding as one of:**

- **Category A — Documentation inaccuracy**: the doc says X but the code does Y. Fix: update the doc.
- **Category B — Code bug not documented**: the code has a defect the docs don't acknowledge. Fix: add a Known Issues entry (and flag for a code fix). *The doc may accurately describe intended behavior while the code silently diverges.*

This distinction matters: Category A findings require doc edits; Category B findings require doc additions plus a code ticket. The report should group findings by category before listing details.

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
- **CRITICAL** — factually wrong, would mislead developers (Category B bugs that silently break behavior are often CRITICAL even if the doc looks plausible)
- **HIGH** — fabricated code, broken diagram rendering, or wrong package names
- **MEDIUM** — partially correct but misleading
- **LOW** — minor naming or completeness issues

---

**Phase 4 — Report & PR**

If `architecture_validation_report.md` exists, append a new pass section. Otherwise create it (lowercase filename).

**Summary Table:**
| Severity | Count | Key Examples |
|----------|-------|-------------|
| CRITICAL | N | ... |
| HIGH | N | ... |
| MEDIUM | N | ... |
| LOW | N | ... |

**Finding Categories:**
Add two tables before the detailed findings — one for Category A (doc inaccuracies) and one for Category B (undocumented code bugs). This makes clear which fixes belong in docs vs which need code tickets.

**Detailed Findings (grouped by file):**
For each doc, list discrepancies with:
- Issue ID: `P{pass}-{severity}{number}` (e.g., P1-H3)
- Finding type: Category A or B
- Description of what's wrong
- **Doc says**: quote with file and line
- **Code shows**: actual source with file:line
- Severity with justification
- Root cause category
- Recommended fix

**What's Accurate:** List what the docs got right — prevents overcorrection.

**Root Cause Analysis:** Tally root causes. Identify the dominant pattern to guide prevention.

**Skill Improvement Recommendations:**
Add a section documenting any validation gaps discovered during this run — types of claims in the docs that had no corresponding check in this skill. For each, record: the gap, the stack it applies to, a proposed mechanical check, and a concrete example from this run. This section drives improvement of the skill itself across future projects.

**Create a PR:**
After writing the report:
1. Create a branch: `git checkout -b validation/pass-{N}-{date}`
2. Stage the report: `git add architecture_validation_report.md`
3. Commit: `git commit -m "docs: add architecture validation report (Pass {N})"`
4. Push: `git push -u origin validation/pass-{N}-{date}`
5. Open a PR targeting `main`:
   - Title: `docs: Architecture Validation Report — Pass {N} ({date})`
   - Body: Summary Table + one-line dominant root cause + Category A vs B breakdown
6. Print the PR URL

If `gh` is unavailable, print the git commands for the user to run manually.

Then ask the user whether to:
1. Review the report in the terminal
2. Fix the issues
3. Fix issues, commit, and update the PR in one step
4. Run another pass with a different focus

### Output Format

Structured markdown report (`architecture_validation_report.md`). Findings grouped by Category A / Category B, then by file, with a summary table and root cause tally at the top of each pass section.

### Guidelines

- Read code, don't infer — every claim must be traceable to a specific file:line
- Verify names exactly — `feeSchedule` vs `feeSchedules` matters; `setupTestServer` vs `mswServer` matters
- Check interfaces verbatim — a component existing doesn't mean its state shape matches the doc
- Count from data — don't trust documented counts; count actual items in actual files
- Imports reveal truth — what a file actually imports tells you its real dependencies
- Grep for hooks, don't assume — the page component probably doesn't use the hook; its child does
- Absent means absent — if monitoring/CI/CD/perf tools aren't in the code, they don't exist
- Check struct JSON tags character-by-character in multi-repo systems — a single leading space silently empties a field
- Run the Mermaid CLI — don't just read the diagram source; broken diagrams are invisible to readers
- Distinguish Category A from Category B — the fix strategy is completely different
- Multi-pass strategy: Pass 1 catches wrong names/shapes/counts/hallucinated components; Pass 2 catches fabricated examples and contradictions; Pass 3 (mechanical) catches phantom diagram nodes, misattributed hooks, non-compiling code blocks, and inter-repo contract drift

---

## Claude Code Setup

To make this skill available as a slash command in Claude Code, create a file at `.claude/commands/validate-architecture.md`:

```markdown
---
description: Validate architecture documentation against the actual source code
---

Validate architecture documentation against the actual source code in this project.

[paste the Phase 0–4 Task prompt above here, replacing {{INPUT}} with $ARGUMENTS]
```

Then invoke with: `/validate-architecture [optional: specific doc or focus area]`

---

## Changelog

| Version | Date | Changes |
|---|---|---|
| 1.1 | 2026-03-19 | Field-tested on 4-repo Go/React system (MDPP). Added Phase 0 (repo discovery + stack detection), stack-specific Phase 1 agents (Go, Python, Java, data pipeline, serverless/IaC), Mermaid CLI rendering check, semicolon-in-label error pattern, inter-repo data contract check (catches silent field-empty bugs from JSON tag whitespace), Category A vs B finding classification, Skill Improvement Recommendations section in report, PR creation step |
| 1.0 | 2026-03-01 | Initial version, derived from PCS-UI 3-pass architecture validation (52 issues across 7 docs) |
