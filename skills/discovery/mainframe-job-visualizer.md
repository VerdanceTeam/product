# Mainframe Job Call-Chain Visualizer

Build an interactive React-based explorer for mainframe COBOL systems from a cross-reference report. The tool parses job-to-program and program-to-program relationships, optionally counts effective lines of code from COBOL source files, and renders each job as a container with its full program call tree inside — including DAG structures where multiple parents call shared subprograms.

Use this skill whenever the user has a mainframe cross-reference report (CSV or spreadsheet) describing jobs, programs, and their relationships and wants to visualize, explore, or understand the system's structure. Also use when the user asks to diagram COBOL batch job flows, analyze program call chains, or build a job dependency explorer.

---

## Inputs

**Required:**
- A cross-reference CSV with columns: `Referred by`, `Referring Object Type`, `Relationship`, `Object Name`, `Object Type`, `Legacy Object`, `Object Description`
- Key relationship types to extract:
  - `"Runs Program"` — Job → Program (direct execution)
  - `"Calls Program"`, `"Links Program"`, `"Xctls to Program"`, `"Loads Program"` and their `"thru Decision"` variants — Program → Program (call chain)

**Optional:**
- A directory/zip of COBOL source files (`.cbl`) for LOC counting
- Job scheduler extract (predecessor/successor pairs) for execution ordering

## Pipeline Overview

```
Cross-Ref CSV → Java Parser → SystemModel → JSON Export → React Viewer
                                  ↑
                           COBOL Sources → LOC Counter (Python) → LOC JSON
```

### Step 1: Parse the Cross-Reference

Write a Java program to ingest the CSV. The data model has three key classes:

- **`Job`** — name + set of direct `Program` references
- **`Program`** — name + set of called `Program` references + optional `linesOfCode` (OptionalInt) and `cyclomaticComplexity` (OptionalInt)
- **`SystemModel`** — registry with `getOrCreateJob(name)` / `getOrCreateProgram(name)` for deduplication

**Parsing rules:**
- Filter to rows where `Referring Object Type` is `"Job"` or `"Program"` and `Object Type` is `"Program"`
- Skip copybooks, macros, data stores, system programs, and assembler files
- Flatten all program-to-program relationship variants (Calls, Links, Xctls, Loads) into a generic "calls" edge
- Strip the `"thru Decision"` suffix when matching relationship types

### Step 2: Deduplicate Jobs

Many mainframe jobs have identical program call chains (same transitive closure of reachable programs). Merge these into single entries with concatenated names (e.g., `"MSSQAB00 / MSSQABSH / MSSQMF2X"`).

**Algorithm:**
1. For each job, compute its **signature**: the sorted set of all transitively reachable program names
2. Group jobs by identical signatures
3. Merge each group into a single job, joining names with ` / `

Typical reduction: ~29% (e.g., 902 → 643 unique jobs).

### Step 3: Count COBOL LOC (if source files provided)

Use Python to count effective lines of code per COBOL program:

```python
# For each .cbl file, count lines that are:
# - Not blank/whitespace-only
# - Not comment lines (column 7 = '*', 'D', 'd', '/')
# - Not compiler directives (EJECT, SKIP1/2/3)
# - Have non-blank content in columns 7-72 (the COBOL code area)
```

Map filenames to program names by stripping the `.cbl` extension (e.g., `H99PD200.cbl` → `H99PD200`). Expect ~90% match rate; unmatched programs are typically system-provided utilities not in the source repo.

Output two files:
- `program_loc.csv` — for the spreadsheet tab
- `program_loc.json` — `{"PROGRAM_NAME": loc, ...}` for embedding in the React viewer

### Step 4: Export to JSON

Export the deduplicated system model as JSON for the React viewer. Structure per job:

```json
{
  "n": "JOB_NAME",
  "d": ["DIRECT_PGM1", "DIRECT_PGM2"],
  "p": ["ALL_REACHABLE_PGM1", "ALL_REACHABLE_PGM2", "..."],
  "e": [["CALLER", "CALLEE"], ["CALLER2", "CALLEE2"]]
}
```

Use short keys (`n`, `d`, `p`, `e`) and `separators=(',',':')` to minimize JSON size. Embed the full JSON array directly in the React component as a constant.

### Step 5: Build the React Viewer

The viewer is a single `.jsx` artifact with embedded data. Key components:

#### Sidebar
- Searchable job list sorted by program count (largest/most complex first)
- Each entry shows: job name, program count, edge count, total LOC
- Search matches against job names (including merged names like `"MSSQAB00 / MSSQABSH"`)

#### Graph View (SVG)
- Pan (drag) and zoom (scroll wheel) via viewBox manipulation
- Each job rendered as its own call tree when selected

#### Layout Algorithm — Layered Graph with Barycenter Crossing Minimization

This is the most important part for readability. The layout must minimize edge crossings.

**Layer assignment:** Longest-path DFS from direct programs (layer 0). Each program is placed at the maximum depth reachable from any root. This ensures children are always below all their parents.

**Crossing minimization:** Barycenter heuristic with alternating sweeps:
1. Start with alphabetical ordering within each layer
2. **Top-down sweep:** For each layer (top to bottom), sort nodes by the average position of their parents in the layer above
3. **Bottom-up sweep:** For each layer (bottom to top), sort nodes by the average position of their children in the layer below
4. After each sweep, count total edge crossings across all adjacent layer pairs
5. Keep the ordering with the fewest crossings seen so far
6. Repeat for up to 24 iterations or until zero crossings

**Crossing count:** For each pair of adjacent layers, collect all edges as `(upperIndex, lowerIndex)` pairs. Two edges cross if and only if `(u1 < u2 && l1 > l2) || (u1 > u2 && l1 < l2)`.

**Nodes without neighbors** in the reference layer keep their current position (don't displace them based on a -1 barycenter).

#### Node Sizing and Coloring (LOC-based)

**Size:** Nodes scale with LOC using square-root scaling for perceptual area proportionality:
```
t = min(loc / ceiling, 1)
s = sqrt(t)
width  = 88 + s * 192    (range: 88px – 280px)
height = 28 + s * 36     (range: 28px – 64px)
```

**Color:** Three-stop gradient from green (low LOC) → yellow (mid) → red (high LOC), computed relative to the ceiling parameter.

**LOC scale slider:** A range input (100–10,000) in the header bar controls the ceiling. Lowering it increases contrast between small and large programs. Both size and color respond to the same slider.

#### Node Types and Visual Encoding
- **Direct programs** (job runs them): Blue fill (`#DBEAFE`), blue stroke, bold label — always blue regardless of LOC
- **Subprograms** (reached via call chains): Green→yellow→red fill/stroke based on LOC. Programs without LOC data default to light green
- **Edges:** Dashed curves (cubic bezier) with arrowhead markers. Highlighted on hover

#### Hover Interaction
When hovering a node, highlight it and all directly connected nodes/edges. Dim everything else to `opacity: 0.25`.

### Step 6: Spreadsheet Output (if LOC data available)

Create an `.xlsx` with a "Program LOC" tab:
- Columns: Program name (monospace font), Effective LOC (number-formatted), In Cross-Reference (Yes/blank)
- Sorted by LOC descending
- Auto-filter and frozen header row
- Summary stats block: total programs, matched count, total/max/median/mean LOC

---

## Key Structural Properties of Mainframe Systems

These properties inform design decisions and are worth communicating to the user:

- **It's a forest, not a chain.** Most jobs (~65%) directly run multiple programs, each with its own call subtree.
- **It's a DAG, not a tree.** Shared utility programs (like SSBCPLAN, Y2KWDATE) are called by multiple parents. The visualization must show convergent edges, not duplicate nodes.
- **Significant duplication.** ~29% of jobs have identical transitive call chains and can be merged. An additional ~14% are strict subsets of larger jobs.
- **Scale varies wildly.** Jobs range from 1 program (single leaf) to 100+ programs with deep branching. The viewer must handle both gracefully via pan/zoom.
- **LOC distribution is long-tailed.** Median ~370 LOC, but outliers exceed 10,000. Square-root scaling with an adjustable ceiling handles this well.

---

## File Structure

```
crossref-diagram/
├── src/main/java/com/crossref/
│   ├── model/
│   │   ├── Job.java
│   │   ├── Program.java
│   │   ├── SystemModel.java
│   │   └── JobDeduplicator.java
│   ├── io/
│   │   ├── CrossReferenceParser.java
│   │   └── D2Exporter.java          # Optional: D2 output alternative
│   ├── Main.java                     # --dedup flag for deduplication
│   └── JsonExporter.java            # JSON output for React viewer
├── target/
│   ├── system.json                   # Full JSON export
│   └── system.min.json               # Minified for embedding
└── pom.xml
```

## Compilation and Execution

```bash
# Compile
javac -d target/classes src/main/java/com/crossref/**/*.java

# Run parser + JSON export
java -cp target/classes com.crossref.JsonExporter input.csv output.json

# Run with D2 output (alternative)
java -cp target/classes com.crossref.Main input.csv output.d2 --dedup
```

## D2 Export (Alternative Output)

If the user wants D2 instead of or in addition to the React viewer, export each job as a D2 container:

```d2
JOB_NAME: {
  label: "JOB_NAME"
  style: { fill: "#EFF6FF"; stroke: "#3B82F6"; stroke-width: 2 }

  DIRECT_PGM: { shape: rectangle; style: { fill: "#DBEAFE"; bold: true } }
  SUB_PGM:    { shape: rectangle; style: { fill: "#F0FDF4" } }

  DIRECT_PGM -> SUB_PGM: { style.stroke-dash: 3 }
}
```

Programs appear once per container. Shared subprograms get multiple incoming edges (DAG preserved). Direct programs are visually distinct (bold, blue). Node width/height scale with LOC when available.
