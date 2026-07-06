---
name: team-report
classification: workflow
classification-reason: "Multi-step orchestration: team composition -> parallel analysis -> HTML report generation -> browser open"
deprecation-risk: none
description: |
  Multi-agent team analysis with interactive HTML report. Dynamically spawns 3-10
  expert agents based on analysis scope, then generates a standalone HTML report
  with paginated navigation and localStorage-based Accept/Defer/Reject decision UI.
  Feedback accumulates in docs/reports/feedback.yaml for progressive improvement.
  Triggers: /team-report, team report, analysis report, HTML report,
    expert analysis, component analysis
  Keywords: team, report, analysis, HTML, expert, localStorage, decision, feedback
argument-hint: "/team-report {analysis topic}"
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - Agent
---

# Team Report — Multi-Agent Analysis with Interactive HTML Report

## Purpose

Parallel expert agent analysis of any codebase aspect, outputting a paginated interactive HTML report with inline Accept/Defer/Reject decision cards. Feedback accumulates in `docs/reports/feedback.yaml` for progressive improvement.

**Difference from bkit report-generator:**
- report-generator: PDCA completion report (Markdown)
- team-report: Multi-agent analysis report (Interactive HTML + localStorage + Feedback Loop)

## When to Use

- Deep analysis of specific codebase aspects (color, architecture, performance, etc.)
- Comparing/analyzing multiple components simultaneously
- Sharing analysis results as visual reports
- Tracking decision-making on recommendations

---

## Workflow

### Phase 1: Topic & Team Composition

1. **Identify analysis topic** — extract target from user request
2. **Determine file scope** — Glob/Grep to find related files
3. **Dynamic team sizing** — determine agent count by scope complexity (min 3, max 10)
4. **Reference feedback** — read `docs/reports/feedback.yaml` for past lessons (same path Phase 6 writes to — keep these identical or accumulation breaks)

**Dynamic Scaling:**

| Scope | Files | Agents | Example |
|-------|-------|--------|---------|
| Small | 1-5 | 3 | Single component analysis |
| Medium | 6-15 | 5 | Route/module analysis |
| Large | 16-30 | 7 | Full UI system analysis |
| Full System | 31+ | 9 | Entire codebase analysis |

Agent count ≈ `ceil(files / 4)`, clamped to 3–10. Last slot is always Visual QA, so specialists = agents − 1. Boundaries are exclusive (16 files → Large, not Medium).

**Team Principles:**
- Each agent gets a non-overlapping file scope
- Last agent is always Visual QA Expert (Playwright screenshots)
- Agent types: code analysis = `Explore`, screenshots = `general-purpose`

> **Orchestration Note — use subagents, NOT Agent Teams.** These experts work independently and report once; they never message each other. That is the subagent profile (foreground parallel `Agent` calls), not Agent Teams. Do NOT use Agent Teams here: `TeamCreate`/`TeamDelete` were removed in Claude Code v2.1.178+, and in-process teammates are NOT restored by `/resume` or `/rewind` — a long report run would lose every teammate mid-flight.

**Team Example (Small — 3):**
```
| Expert               | Subagent Type   | Scope                   |
|----------------------|-----------------|-------------------------|
| Code Analysis Expert | Explore         | Target files (all)      |
| Architecture Expert  | Explore         | Related configs, layouts|
| Visual QA Expert     | general-purpose | Playwright screenshots  |
```

**Team Example (Medium — 5):**
```
| Expert               | Subagent Type   | Scope                   |
|----------------------|-----------------|-------------------------|
| Style Expert         | Explore         | SCSS, styles            |
| Component Expert A   | Explore         | Components 1-5          |
| Component Expert B   | Explore         | Components 6-10         |
| Architecture Expert  | Explore         | Layouts, routes, configs|
| Visual QA Expert     | general-purpose | Playwright screenshots  |
```

**Team Example (Large — 7):**
```
| Expert               | Subagent Type   | Scope                   |
|----------------------|-----------------|-------------------------|
| Code Review Lead     | Explore         | Entry files, index      |
| Style & Theme Expert | Explore         | CSS, SCSS, theme        |
| Component Expert A   | Explore         | Components 1-10         |
| Component Expert B   | Explore         | Components 11-20        |
| Router & State Expert| Explore         | Routes, store, state    |
| Architecture Expert  | Explore         | Config, layout, build   |
| Visual QA Expert     | general-purpose | Playwright screenshots  |
```
(Full System — 9: add Backend/API + Hooks/Utils experts, same pattern. Last slot stays Visual QA.)

### Phase 2: Parallel Analysis

<!-- @import(docs/agent-prompts.md) — Load for prompt templates and failure recovery -->

1. **Launch all agents** — single message with N Agent calls, **foreground parallel** (do NOT use `run_in_background`). Foreground returns every result within the parent turn; background leaves only async handles that vanish on session resume (see Orchestration Note above)
2. **Each prompt must include:** absolute file paths, output format, "Thoroughness: very thorough"
3. **Show results summary** (foreground returns all at once — report final status, not live progress):

```
| Expert         | Status | Scope              |
|----------------|--------|--------------------|
| Style Expert   | Done   | app.scss, comps    |
| Arch Expert    | Done   | configs, layouts   |
| Visual QA      | Failed | Screenshots        |
```

4. **Collect all results** — if an agent fails, continue with others and mark section incomplete

### Phase 3: HTML Report Generation

<!-- @import(docs/html-template.md) — Load for HTML/CSS layout spec -->
<!-- @import(docs/diagram-guide.md) — Load when rendering architecture diagrams -->

Generate paginated HTML report:
```
Page 1.  Analysis Team        — team composition
Page 2.  Executive Summary    — stat cards + key findings
Page 3~N. Detailed Analysis   — per-expert results + inline recommendations
Page N+1. Gap Analysis        — comparison table + inline recommendations
Page N+2. Visual Impression   — screenshots (Visual QA Expert)
Page N+3. Proposed Solution   — concrete fixes
Page N+4. Priority Matrix     — implementation priorities
Page N+5. Decision Summary    — aggregate decisions + export
```

Key rule: inline `.inline-rec` cards go **right after the evidence text**, not in a separate page.

### Phase 4: Inline Decision UI

<!-- @import(docs/decision-ui.md) — Load for decision card HTML/JS/CSS spec -->

Each recommendation becomes an Accept/Defer/Reject card with localStorage persistence. Sidebar tracks progress in real-time.

### Phase 5: Output & Open

**Output Path Resolution:**

```
1. TEAM_REPORT_OUTPUT env var (user override)
2. svelte.config.js exists → static/mockup/{slug}.html
3. next.config.js exists   → public/mockup/{slug}.html
4. vite.config.js exists   → dist/mockup/{slug}.html
5. Fallback               → docs/reports/{slug}.html
```

**File Write:**
1. Detect output path using resolution order above
2. Create directory if needed (`mkdir -p`)
3. Write the complete standalone HTML file (all CSS/JS inline)
4. File must be self-contained (only Google Fonts CDN external)

**Browser Preview:**
1. If framework dev server detected (check ports 80, 3000, 5173, 8080):
   - Use Playwright `browser_navigate` to open via localhost
2. If no dev server found:
   - Use `Bash: start "" "{absolute-path}"` to open file directly
3. If Playwright unavailable:
   - Output: `"Report saved to: {absolute-path}. Open manually in browser."`

**Error Handling:**
- Write failure → report error with path, suggest alternative location
- Browser open failure → always fall back to file path output
- Never silently fail — always confirm write success or report error

**Result Summary:**
Output 3-5 line summary of key findings after file is written.

### Phase 6: Feedback Collection

<!-- @import(docs/feedback-system.md) — Load for YAML spec and collection methods -->

Feedback accumulates in `docs/reports/feedback.yaml`:
- Decisions from localStorage (Playwright collection or manual export)
- User direct feedback via conversation
- Lessons learned auto-categorized by topic

**PDCA Integration (Optional):**

```
1. No .bkit-memory.json             → Standalone mode (default)
2. .bkit-memory.json + --pdca-phase → PDCA mode, use the given phase
3. .bkit-memory.json, no flag       → PDCA mode, auto-detect feature/phase from the file
```

**Storage by mode** (`docs/reports/feedback.yaml` is always the source of truth):
- Standalone → write `docs/reports/feedback.yaml` only
- PDCA mode  → write `docs/reports/feedback.yaml` first, THEN append the same decisions to `.bkit-memory.json` (never overwrite; on rec-id conflict, feedback.yaml wins)

---

## CSS Reference

<!-- @import(docs/css-reference.md) — Load for full CSS class catalog -->

See `docs/css-reference.md` for the complete list of required CSS classes.

---

## Example Invocations

```
/team-report color theme analysis
/team-report lobby component architecture review
/team-report performance optimization report
/team-report feedback color-theme — code block font too small
```
