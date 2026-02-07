# Firebase Studio Prompt — Rule Automation Studio (React + MUI)

Copy/paste everything below into Firebase Studio.

---

Build a **static web app** named:

**Rule Automation Studio — Translation Journey (Evaluator Prototype)**

## Hard constraints (must follow)
- **React + TypeScript + Vite**
- Use **Material UI (MUI)** for UI components and styling
- **No backend**, no auth, no external APIs, no secrets. Everything runs **in-browser**.
- Deterministic parsing only (regex + heuristics). **Do NOT call any AI services**.
- `npm run build` must produce a **`dist/`** folder deployable to **Firebase Hosting**.

## What the evaluator must see (core value)
The evaluator must be able to paste a workflow text and click **Run Translation** and then see the full “translation journey” with intermediate outputs, not just the final code.

## UI requirements (MUI)
Use MUI to create a professional layout.

### AppShell
- `AppBar` with:
  - Title: “Rule Automation Studio”
  - Subtitle: “Workflow → Rules, with traceability”
  - Buttons: `Load Sample`, `Run Translation`, `Reset`
- `Container` max width `lg`

### Main layout (3-column on desktop, stacked on mobile)
Use `Grid`:

1) **Left panel**: `Paper` containing:
- `TextField multiline` for Workflow Text (monospace toggle optional)
- `TextField` for optional Rule Name override
- Small helper text: “Sanitized content only”

2) **Middle panel**: `Paper` containing a **Stepper** (vertical preferred)
Stages (must be exactly these, with icons):
1. Normalize
2. Legend
3. Entities (tables/prefixes)
4. Steps & Decisions
5. Patterns
6. Codegen

Each step shows:
- A one-line explanation (“what we do here”)
- A compact output preview (list/table/short JSON)
- `Chip` status: ✅ Complete or ⚠️ Warning (e.g. “Legend not found”)

3) **Right panel**: `Paper` containing `Tabs`:
- Tab 1: **Summary** (human-readable)
- Tab 2: **Analysis JSON** (pretty printed, scrollable, code font)
- Tab 3: **Generated TypeScript** (scrollable, code font)

### Actions bar
Under tabs, add MUI buttons:
- Download `analysis.json`
- Download `GeneratedRule.ts`
- Download `journey.md` (a readable report of each stage)
- Copy “Share Summary” to clipboard

### Trace mode
Add a `Switch` “Show trace”. When enabled, show an accordion list of trace events per stage:
- which regex/rule fired
- a short input snippet
- a short output snippet

## Data model contract (must implement)
Create a single output object:

```ts
type JourneyStageId =
  | "normalize"
  | "legend"
  | "entities"
  | "steps"
  | "patterns"
  | "codegen";

type JourneyTraceEvent = {
  stage: JourneyStageId;
  rule: string;
  inputSample?: string;
  outputSample?: string;
  notes?: string;
};

type Analysis = {
  title?: string;
  domain?: string;
  normalizedText: string;
  legend: Array<{ abbr: string; meaning: string }>;
  detectedTables: string[];
  tablePrefixes: Array<{ prefix: string; count: number }>;
  steps: Array<{ id: string; text: string; type: "action" | "decision" }>;
  decisionEdges: Array<{ from: string; to: string; label?: "YES" | "NO" | "NEXT" }>;
};

type Patterns = {
  naming: { ruleClassName: string; fileName: string };
  intent: string;
  dataAccessPlan: { reads: string[]; writes: string[] };
  confidence: number; // 0–100
};

type JourneyOutput = {
  analysis: Analysis;
  patterns: Patterns;
  generatedTs: string;
  summaryText: string;
  journeyMd: string;
  trace: JourneyTraceEvent[];
};
```

## Deterministic parsing logic (no AI)
Implement these stages with regex + heuristics and generate trace events.

### 1) Normalize
- trim, normalize whitespace
- attempt to detect `Title:` and `Domain:`
- output `normalizedText`

### 2) Legend extraction
- detect section headers: `Legend`, `LEGEND`
- parse lines like:
  - `ABC = meaning`
  - `ABC: meaning`
- stop at blank line or next header-like line

### 3) Entities (tables/prefixes)
- extract uppercase underscore tokens >= 6 chars:
  - regex example: `\b[A-Z][A-Z0-9]{1,10}_[A-Z0-9_]{3,}\b`
- dedupe + sort
- prefixes: substring up to first `_` (include underscore, e.g. `MAP_`)
- count prefixes and show as a small table

### 4) Steps & decisions
- split steps via:
  - numbered: `1.`, `2)`, `Step 1:`
  - bullets: `-`, `*`
- decision detection: lines containing `IF`, `ELSE`, `WHEN`, `THEN`, `?`, `APPROVE`, `REJECT`
- build a simple graph:
  - sequential NEXT edges
  - if IF/ELSE patterns present, add YES/NO edges where reasonable

### 5) Patterns
- infer class name from Title or first meaningful line → PascalCase
- infer intent using keywords (validate, route, calculate, create task, update, etc.)
- reads = detected tables
- writes detected if contains `UPDATE|INSERT|CREATE|SAVE|SET`
- confidence scoring:
  - +20 title/domain present
  - +20 legend present
  - +20 tables >= 3
  - +20 steps >= 5
  - +20 decisions >= 1
  - clamp 0–100

### 6) Codegen
Generate a TypeScript scaffold:
- header comment listing legend/tables/prefixes
- class `${ruleClassName}`
- method `execute(params)` with TODO blocks per step/decision
- no proprietary imports, no DB calls

## Sample workflow
Include a built-in sample workflow showing:
- title + domain
- legend with 5 abbreviations
- 8–12 steps
- 2 decisions
- 8–15 tables from multiple prefixes

## Deliverables
- Full runnable project
- `README.md` with:
  - local run steps
  - Firebase Hosting deployment steps (public dir = `dist`)
- Scripts in `package.json`: `dev`, `build`, `preview`

## Additional polish
- Use MUI theme (dark mode default)
- Error handling: never blank screen; show `Alert` with friendly message
- If a stage finds nothing, show “⚠️ Not found” plus tips
- Buttons enable/disable properly
- Provide `journey.md` export explaining each stage and findings

Build the complete project now.
