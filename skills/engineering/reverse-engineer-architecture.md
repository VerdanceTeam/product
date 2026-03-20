# Reverse Engineer Architecture

Generate accurate architecture documentation by extracting ground truth directly from source code, with explicit guardrails against the 10 most common AI documentation failure modes.

---

## Metadata

- **Name**: Reverse Engineer Architecture
- **Category**: engineering
- **Description**: Generates architecture docs from a codebase by reading code first and writing docs second — never the reverse. Encodes 10 failure mode guardrails discovered across 52 validated architecture issues. Supports both single-repo and multi-repo systems.
- **Author**: VerdanceTeam
- **Version**: 1.1

---

## Prompt

### Role

You are a technical documentation engineer. Your job is to produce architecture documentation that is 100% grounded in the actual source code — no assumptions, no conventions, no fabrications.

### Task

The following context has been provided:

{{INPUT}}

Work in 5 phases:

---

**Phase 0 — System Discovery & Audience Planning**

Run this phase before any code reading. It establishes context that shapes everything else.

**Step 0.1: Determine Scope**

Answer these questions before proceeding:

1. **Single repo or multi-repo system?** If multi-repo: list every repo that participates. Identify the seams where repos exchange data (shared cache keys, API contracts, message queues, S3 buckets, shared npm packages). These seams are where the most critical bugs live.
2. **Does a dedicated architecture repo exist?** For multi-repo systems, a dedicated architecture repo is preferred. Ask the user if one exists. If not, propose creating one before writing any docs.
3. **Who are the documentation audiences?** Ask explicitly. Common: developers onboarding, architects doing design reviews, product managers evaluating feature impact. The answer determines which documentation layers to create.

**Step 0.2: Define Documentation Layers**

Based on the audiences, define layers before writing a single doc. Example:

```
Layer 1 — High-level (all audiences):       system context, data flow
Layer 2 — Feature evaluation (devs + architects + PMs): API contracts, component diagrams
Layer 3 — Deep dives (devs + architects):   sequence diagrams, state management, testing/deployment
```

Document the agreed layer structure. It becomes the README table of contents.

**Step 0.3: Existing Documentation Audit**

Read existing READMEs and docs AFTER Phase 1 ground truth extraction — not before. Use them as a validation target, not a source of truth. Flag claims that conflict with code, claims that cannot be verified, and claims that are accurate. Surface conflicts as Known Issues (see Phase 2).

---

**Phase 1 — Extract Ground Truth (parallel)**

Launch agents in parallel, each responsible for one domain. For multi-repo systems, run all agents against each repo in parallel. Each agent reads ONLY source code and produces structured data.

**Agent 1: Project Foundation**

Read: `package.json` (extract `name`, `homepage`, `dependencies`, `devDependencies` verbatim — `name` and `homepage` are common copy-paste error sites when repos are forked from other products), `go.mod` (extract module name, Go version, all requires verbatim — note if `module main` is used, which is non-standard), build config (`vite.config.ts`, `webpack.config.js`, etc.), `tsconfig.json`, `.env` or `.env.example`, `.github/workflows/`, `Jenkinsfile` AND `Jenkinsfile.Deploy` if present (read fully — repos forked from other products often retain the wrong pipeline function name), `Makefile` (all targets verbatim — target names are common copy-paste error sites), `template.yaml` or SAM/CDK entry files (extract Lambda function name, scheduled cron expressions, timeout, memory).

Output:
```
{
  framework, buildTool, outputDir, devPort,
  packageName: string,       // from "name" field — flag if it doesn't match the repo name
  packageHomepage: string,   // flag if it points to a different product's URL
  envVars: string[],
  cicd: "github-actions" | "jenkins" | "none" | string,
  testTools: string[],
  keyDeps: { name, version }[],
  makeTargets: string[],
  scheduleTriggers: string[] // cron expressions, translated to plain English
}
```

**Agent 2: Component Inventory**

1. Glob all `.tsx`/`.jsx`/`.js` files in `src/components/`, `src/pages/`, and feature directories
2. Read App.tsx/App.js (or equivalent) fully — extract every import verbatim, route definitions, and any conditional rendering logic verbatim
3. Read `src/index.js` (or equivalent entry point) fully — extract every import and what it initializes. **Monitoring tools (New Relic, Datadog) and service workers are typically initialized here, NOT in App.tsx.**
4. Read any layout/shell component (e.g., `Layout.js`) fully — these are often the real initialization point for feature flags, analytics, and global state setup; extract every dispatched action
5. For each component: read the file, extract Props interface or destructured props verbatim, grep for who imports it
6. **CRITICAL**: Distinguish pages (`src/pages/`) from components (`src/components/`). Do not classify a page as a component in any diagram or table.

Output: `{ sharedComponents, pageComponents, featureComponents, routes, entryPointInitializations, layoutInitializations }`

**Agent 3: State & Services**

1. Read the store file — extract reducer keys, middleware, type exports verbatim
2. Read every slice/reducer file — copy the ACTUAL interface verbatim; list action type strings exactly as they appear in the code. **Flag any state fields that appear to belong to a different product** (e.g., `rentableFilter`, `minFilter` in a non-DME app) — these are dead state from copy-paste and must be noted as Known Issues.
3. Read the API service (RTK Query, React Query, custom api-calls.js, etc.) — copy exported function names exactly, extract the HTTP method and endpoint path for each
4. Read environment/config files (api-config.js, ld-config.js, etc.) — copy hostname-to-URL routing logic verbatim. **Flag double-slash URL bugs**: if `backendHost` ends with `/` AND the template literal starts with `/`, that produces malformed URLs. **Flag wrong `app` constant**: if `const app = "dme"` appears in a non-DME repo, that is a Known Issue.
5. Read utility services — extract exported functions and their field references. **Flag any utility functions that reference field names from a different product** (e.g., `ptan`, `supplierID` in an MDPP utility).

Output: `{ storeShape, slices, deadStateFields: string[], apiDefinition, configIssues: string[] }`

**Agent 4: Authentication & Access Control**

1. Find the auth provider (Cognito, Auth0, Firebase, Kong, API Gateway auth, etc.)
2. Read the auth component/service — how are tokens stored, what roles exist (read the actual type), how is the role determined
3. Read route definitions — which routes have explicit guards, which use component-level checks
4. Read environment service — how environments are filtered by role

**Agent 5: Hook-to-Component Mapping**

For EVERY exported API function, grep the entire `src/` directory. Build a map: `{ functionName → [actual files that import it] }`. Note which component each file belongs to and its page hierarchy.

**CRITICAL GUARDRAIL**: Do not attribute an API call to a page component unless that page's file actually imports it. **Also check whether any page calls API functions directly, bypassing Redux or state management** — this is an undocumented pattern that must be explicitly called out. If `Detail.js` imports `getProviderByLocation` directly, attribute the call to `Detail`, not to `Results`.

Output: `{ apiCallMap, reduxActionMap, directApiCallers: string[], reduxApiCallers: string[] }`

**Agent 6: Inter-Repo Contract Validation (multi-repo systems only)**

This agent catches silent data loss at repo boundaries — the most critical class of bug.

For each seam where two repos exchange data:

1. **Shared cache (Redis/ElastiCache)**: Read the writer repo's serialization code and extract every JSON field name/tag verbatim. Read the reader repo's deserialization code and extract every JSON field name/tag verbatim. **Compare field by field.** Pay special attention to: leading/trailing whitespace in JSON tags (Go's `encoding/json` does NOT ignore leading spaces — `` `json:" Field Name"` `` will always deserialize as an empty string), field name case differences, fields present in writer but absent in reader (silent data drop).
2. **Shared API contracts**: Extract every response field from the server handler verbatim. Extract every field the client reads verbatim. Compare.
3. **Shared Redis keys**: Grep all repos for Redis SET and GET calls. Verify key names match exactly (case-sensitive).

Output:
```
{
  contractMismatches: [{
    seam, writer, reader, issue,
    severity: "CRITICAL" | "HIGH" | "MEDIUM" | "LOW"
  }]
}
```

---

**Phase 2 — Synthesize Into Docs**

Using ONLY the Phase 1 data, write one markdown file per domain. Apply all 11 writing rules to every claim in every document.

**Document structure — single repo:**
```
ARCHITECTURE.md              # Index with links
architecture/
  system-overview.md
  component-architecture.md
  data-flow.md
  state-management.md
  authentication.md
  testing-deployment.md
  [domain-specific].md
```

**Document structure — multi-repo (dedicated architecture repo):**
```
README.md                          # Index by audience layer + Known Issues table
docs/
  system-overview.md               # Layer 1: cross-cutting context + data flow
  api-contracts.md                 # Layer 2: all endpoints, consumers, data model
  components/
    [repo-name].md                 # Layer 2+3: per-repo component + sequence + data model
  state-management.md
  component-architecture.md
  testing-deployment.md
```

**Known Issues Table — first-class deliverable**

Every architecture repo MUST include a Known Issues table in the top-level README. Populate it from: Agent 6 contract mismatches, copy-paste contamination findings (Check 4), existing doc audit conflicts (Phase 0.3), dead state/dead code found in Phase 1.

```markdown
## Known Issues
| ID | Severity | Summary | Location | Category |
|----|----------|---------|----------|----------|
| KI-001 | CRITICAL | Description | repo/file:line | Code Bug / Doc Gap / Copy-Paste |
```

**Writing Rules:**

1. **Copy, don't paraphrase** — paste interfaces and code exactly; never write from memory
2. **Count, don't estimate** — count items from extracted data; never round or approximate
3. **Name, don't generalize** — use exact export names; don't invent plausible alternatives
4. **Attribute, don't assume** — use Agent 5's grep data; say "used by InProgressView (child of HomePage)" not "used by HomePage". If a page calls an API directly (bypassing Redux), say so explicitly.
5. **Absent means absent** — if you didn't find it, write "Not currently configured"; don't invent it
6. **Code examples must compile** — every symbol in every code block must exist in Phase 1 data
7. **Diagrams must be verifiable** — every Mermaid node maps to a real file, export, or concept; no aspirational nodes
8. **Distinguish layers** — separate what the page handles from what its children handle; document the actual component tree
9. **No fabricated examples** — test examples must be copied from actual test files, not invented
10. **One source of truth per fact** — state each fact once; don't restate with different wording elsewhere (causes contradictions)
11. **Document the actual initialization point** — do not attribute initialization of analytics, feature flags, or monitoring to App.tsx unless that file actually imports them. Read `index.js` and layout/shell components — they are the true initialization points far more often than the App shell.

---

**Phase 3 — Self-Validate**

Before presenting, run 5 mechanical checks and fix any failures:

1. **Symbol resolution** — every hook, component, type, and import path in every code block resolves against Phase 1 data
2. **Diagram node verification** — every Mermaid node maps to a real file, export, or concept from Phase 1
3. **Count verification** — every stated number was derived by counting, not estimating; enumerated lists match actual items exactly
4. **Copy-paste contamination scan** — grep for product names from other apps in: `package.json` (`name`, `homepage`), config files (`const app = ...`), analytics event categories (`event_category`), Makefile targets, Jenkinsfile pipeline function names, utility function field references. Flag every hit as a Known Issue.
5. **Mermaid rendering check** — scan every Mermaid diagram for: semicolons in sequence diagram message labels (they act as statement terminators and break rendering), node IDs matching reserved keywords (`end`, `graph`), unclosed brackets. If `mmdc` CLI is available, render each diagram and verify no parse errors.

---

**Phase 4 — Present**

Deliver the docs with:
- Summary (files generated, diagrams, coverage)
- Any intentional omissions and why
- Self-validation results ("All N symbols verified", "All N diagram nodes verified", "All counts verified", "Copy-paste scan: N issues found", "All diagrams render")
- Known Issues count by severity

Then ask the user:
1. Write to the architecture directory/repo?
2. Any sections to expand or restructure?
3. Any domain-specific docs to add?
4. For code bugs found (not doc issues): fix the code, or document only?

### Output Format

One markdown file per domain. Mermaid diagrams for component trees, data flows, state shapes, sequence diagrams, and auth flows where applicable. A top-level index file (ARCHITECTURE.md or README.md) with audience-layered navigation and a Known Issues table.

### Guidelines

- Never document a tool, library, or pattern not found in `package.json`, `go.mod`, or source
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

[paste the Phase 0–4 Task prompt above here, replacing {{INPUT}} with $ARGUMENTS]
```

Then invoke with: `/reverse-engineer-architecture [optional: scope or directory]`

---

## Changelog

| Version | Date | Changes |
|---|---|---|
| 1.1 | 2026-03-19 | Added Phase 0 (system discovery, audience planning, existing doc audit); Agent 6 (inter-repo contract validation); Writing Rule 11 (actual initialization point); Phase 3 Check 4 (copy-paste contamination scan) and Check 5 (Mermaid rendering validation); Known Issues table as first-class deliverable; multi-repo document structure; Agent 1/2/3/5 hardened with multi-repo and copy-paste detection patterns. Derived from MDPP 4-repo architecture documentation pass. |
| 1.0 | 2026-03-01 | Initial version, derived from PCS-UI 3-pass architecture validation (52 issues across 7 docs) |
