# Firebase Studio Prompt — Rule Automation Studio (Agent Execution Workflow)

Copy/paste everything below into Firebase Studio.

---

Build a web app called:

**Rule Automation Studio — Agent Execution Workflow (Evaluator Prototype)**

## 0) What this prototype must demonstrate
This is an evaluator demo that visualizes the end-to-end **multi-agent translation pipeline**:

1) User uploads a **Visio file** (`.vsdx` or `.vsd`)
2) **Agent A: Visio→PDF Converter** produces a PDF artifact
3) **Agent B: PDF→Images Converter** produces page images
4) **Agent C: Vision Extractor** takes images and produces **Vision JSON** (diagram semantics, legend, tables, steps, decisions)
5) **Agent D: JSON Analyzer** creates structured analysis + detected table prefixes + a decision graph
6) **Agent E: RAG Pattern Learner** retrieves “closest existing rules”, learns patterns, and produces a pattern summary
7) **Agent F: TypeScript Rule Generator** generates a TS rule scaffold
8) **Agent G: Validator/Compiler** runs validation + compile checks and produces a result report

### Important: static evaluator prototype
This must be a **static, client-side prototype**. It must **NOT** call external APIs.

Therefore, some stages will be **SIMULATED** but must still look realistic and traceable:
- Visio→PDF and PDF→Images: simulated artifact generation
- Vision API: simulated by producing a realistic Vision JSON payload (and allow user to paste their own Vision JSON)
- RAG: simulated using a small in-app “existing rule library” dataset + simple scoring
- Validate/compile: simulated checks with a compile-style report (PASS/WARN/FAIL)

Every simulated stage must show a clear **SIMULATED** badge and a note: “In production this is done by …”

## 1) Hard constraints (must follow)
- **React + TypeScript + Vite**
- Use **Material UI (MUI)**
- **No backend**, no auth, no secrets
- Runs fully **in the browser**
- Must build to `dist/` and be deployable to **Firebase Hosting**

## 2) UI: make the pipeline visible (this is the core)
Use MUI to create a product-quality experience.

### AppShell
- `AppBar` with title “Rule Automation Studio” and subtitle “Agent execution workflow + artifacts”.
- Buttons: `Run Pipeline`, `Run with Sample Data`, `Reset`.

### 3-panel layout (desktop), stacked on mobile
Use `Grid`:

#### Left: Inputs (Paper)
- Upload control for **Visio file** (`.vsdx/.vsd`). Store in memory only. Show filename/size.
- Optional upload controls:
  - PDF upload (optional)
  - Images upload (optional)
- Textarea: **Vision JSON override** (optional). If provided, Agent C uses this instead of simulated Vision output.
- Input field: optional Rule Name override.

#### Middle: Agent Pipeline (Paper)
Show a **vertical Stepper** (or node graph if you prefer) containing these 8 stages, exactly:
- Agent A: Visio→PDF Converter
- Agent B: PDF→Images Converter
- Agent C: Vision Extractor
- Agent D: JSON Analyzer
- Agent E: RAG Pattern Learner
- Agent F: TypeScript Rule Generator
- Agent G: Validator/Compiler

Each stage row must show:
- status chip: Pending / Running / Completed / Warning / Failed
- duration
- a “SIMULATED” badge if applicable

Clicking a stage selects it and updates the right panel content.

#### Right: Stage Details (Paper)
Tabs:
- **Input** (what this stage consumed)
- **Output** (what this stage produced)
- **Logs** (timestamped logs, like a job console)
- **Artifacts** (download links for files produced)

Also include:
- A top-level “Run timeline” panel listing all stage events with timestamps.
- A top-level “Artifacts Explorer” listing all produced artifacts.

## 3) Execution behavior (must feel like a real run)
When the user clicks **Run Pipeline**:
- Execute stages sequentially with small delays (300–900ms) so progress is visible.
- Append logs continuously.
- Each stage produces an output object and optionally an artifact.

### Artifacts that must exist
Provide downloadable artifacts (as Blob downloads) with these names:
- `workflow.pdf` (simulated)
- `page-1.png`, `page-2.png` (simulated; generate via canvas with “Page 1/2” text)
- `vision.json`
- `analysis.json`
- `rag-results.json`
- `GeneratedRule.ts`
- `compile-report.json`
- `journey.md`

## 4) Simulated Vision JSON (realistic shape)
Create a realistic `vision.json` structure, for example including:
- `title`, `domain`
- `legend` array (abbreviation → meaning)
- `detectedTables` array
- `steps` array (id, text, type action/decision)
- `edges` array for decision flow

If user provides Vision JSON override, validate it lightly and use it.

## 5) Simulated RAG (local rule library)
Include a small in-app dataset of 8–12 “existing rules”, each with:
- ruleName
- domain (AUTH/APPEAL/etc.)
- referenced tables
- short code snippet

Implement in-browser retrieval scoring based on:
- keyword overlap with steps
- table prefix overlap
- domain match

Return top 3 matches with scores and explain “patterns learned” (naming, common tables, common step types).

## 6) TypeScript scaffold generation
Generate `GeneratedRule.ts` with:
- header comment listing detected legend + tables + prefixes
- class name derived from title/rule name
- `execute(params)` method
- TODO blocks per extracted step
No proprietary imports and no DB calls.

## 7) Validator/Compiler simulation
Create `compile-report.json` with checks like:
- class name is valid
- file name matches rule name
- no forbidden identifiers
- basic “balanced braces” check
- TODOs present

Report overall: PASS/WARN/FAIL.

## 8) README (required)
Provide `README.md` with:
- local run steps
- build steps
- Firebase Hosting deploy steps (public dir = `dist`)
- a section “What is simulated vs production”

## 9) Strong instruction to the generator
Do NOT build a single “parse workflow text” tool. This must be a **multi-stage agent pipeline visualizer** with 8 explicit stages and per-stage **logs + artifacts + timeline**.

Build the complete project now.
