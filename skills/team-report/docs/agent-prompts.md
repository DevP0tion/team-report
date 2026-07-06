---
id: agent-prompts
phase: [2]
depends: []
load-when: "dispatching agents in Phase 2"
created: 2026-04-01
---

# Agent Prompt Templates

## Dispatch Rules

1. All agents launch in a **single message**, **foreground parallel** (do NOT use `run_in_background`) — foreground returns each result in the parent turn; background agents leave only async handles that are lost on session resume
2. Each agent gets a **non-overlapping file scope** — no scope overlap
3. Last agent is always **Visual QA Expert** (Playwright screenshots)
4. Agent types: code analysis = `Explore`, screenshots = `general-purpose`

## File Scope Assignment (Phase 1 → 2)

Partition the Glob/Grep file list so no file lands in two agents:
1. Group by kind: styles (`.css/.scss`), components (`.tsx/.svelte/.vue`), logic/config (`.ts`/config/routes)
2. One specialist per group; split a large group round-robin (Component A = files 1–N, Component B = N+1–2N)
3. Visual QA gets **no files** (browser only)
4. Validate: every file appears in exactly one agent's scope; flag any leftover

## Agent Type & Model Selection

| Need | Subagent Type | Model |
|------|---------------|-------|
| Code/architecture analysis (read files) | `Explore` | inherit lead (use `opus` only for deep analysis) |
| Screenshots / browser / simple extraction | `general-purpose` | `sonnet` or `haiku` |

- `Explore` is read-only (Glob/Grep/Read) — correct for analysis, and it *can't* edit, which is what we want.
- Set a per-agent `model` only to **downgrade** simple work; otherwise inherit the lead model. Do not over-fragment models.

## Variable Resolution

Resolve every `{VAR}` before injecting into a prompt:

| Variable | Source | Mandatory |
|----------|--------|-----------|
| `{EXPERT_ROLE}` | agent's role | yes |
| `{ABSOLUTE_FILE_PATHS}` | Phase 1 scope assignment (absolute) | yes |
| `{CONTEXT_FILES}` | related read-only files | no |
| `{ANALYSIS_TOPIC}` | user request, normalized 2–5 words lowercase | yes |
| `{PAST_LESSONS_IF_RELEVANT}` | docs/reports/feedback.yaml → active lessons | no |

If an optional var is empty, omit its line entirely — never inject "N/A".

## Base Template (All Agents)

```
You are a {EXPERT_ROLE} analyzing a codebase section.

## Analysis Scope
Files to analyze:
{ABSOLUTE_FILE_PATHS}

Related context (read-only, do not analyze):
{CONTEXT_FILES}

Topic: {ANALYSIS_TOPIC}
Historical lessons: {PAST_LESSONS_IF_RELEVANT}

## Output Format
Respond in this exact structure:

### Findings
- Finding 1: {description} ({file}:{line})
- Finding 2: ...
(3-10 findings, each with specific file path + line number)

### Quantitative Data
| Metric | Value |
|--------|-------|
(Include measurable data: # instances, lines affected, % coverage)

### Recommendations
R{N}. {Title}
  - Priority: {high|medium|low}
  - Description: {2-3 sentences with concrete action}
  - Files: {affected files}

## Thoroughness: Very Thorough
- Target 5-8 key issues/patterns (not every minor detail)
- Provide specific file paths + line numbers
- Quantify: # instances, lines affected, % coverage (e.g. "4/8 components")
- Propose concrete solutions, not general advice
- Keep output under ~32,000 chars (TASK_MAX_OUTPUT_LENGTH default) — compress to the most important findings; overflow is silently truncated to a disk file
```

## Code Analysis Expert (Explore)

> Extends Base Template — also include its **Thoroughness: Very Thorough** block.

```
You are a Code Quality Expert reviewing {COMPONENT_NAME} components.

Files to analyze (absolute paths):
- {FILE_1}
- {FILE_2}
- {FILE_3}

Analysis focus:
1. Code patterns & duplication
2. API inconsistencies (props, events, types)
3. TypeScript compliance
4. Performance concerns (unnecessary re-renders, memory leaks)

Output sections:
### Code Patterns
(Find 3-5 repeated patterns with impact assessment)

### API Analysis
| Component | Prop | Type | Consistent? |
(Table comparing prop patterns across components)

### Recommendations
R{N}. {Title}
  - Priority: high|medium|low
  - Description: ...
  - Files: ...
```

## Architecture Expert (Explore)

> Extends Base Template — also include its **Thoroughness: Very Thorough** block.

```
You are an Architecture Expert reviewing project structure.

Files to analyze:
- {CONFIG_FILES}
- {LAYOUT_FILES}
- {ROUTE_FILES}

Analysis focus:
1. Layer separation and dependency direction
2. Naming conventions and file organization
3. Configuration consistency
4. Import graph complexity

Output sections:
### Architecture Overview
(Describe current structure with layer diagram data)

### Consistency Issues
| Area | Expected | Actual | Severity |
(Table of inconsistencies)

### Recommendations
R{N}. {Title}
  - Priority: high|medium|low
  - Description: ...
```

## Visual QA Expert (general-purpose)

```
You are a Visual QA Expert using Playwright to capture and assess the UI.

Task:
1. Navigate to: {REPORT_URL_OR_APP_URL}
2. Take screenshots of 3-5 key pages/states
3. Assess: layout alignment, readability, color contrast, interaction feedback
4. Test interactions: buttons, forms, navigation

Use the browser MCP tools available in your environment:
- browser_navigate to open the URL
- accessibility snapshot (browser_snapshot, or browser_get_page_info depending on the MCP) to read structure
- browser_take_screenshot (or browser_screenshot) for visual evidence
- set a ~30s navigation timeout so a slow load can't hang the agent

Output sections:
### Visual Impression
- Layout: {assessment with screenshot references}
- Readability: {font sizes, contrast ratios}
- Interactivity: {working | broken | incomplete}

### Recommendations
R{N}. {Title}
  - Priority: high|medium|low
  - Description: ...
```

## Agent Failure Recovery

If an agent fails (timeout, error, no output):

**Gate before rendering:** if fewer than 1/3 of agents succeeded, abort and ask the user to narrow scope or retry. After dropping a failed section, **renumber pages so `data-page` is gap-free** — `totalPages` derives from the DOM, so the sidebar and counter stay in sync automatically.

1. **Continue** with all successful agents — do NOT abort the report
2. Mark the failed agent's section in HTML as incomplete:
   ```html
   <div class="page" data-page="N" data-title="[INCOMPLETE] Style Analysis">
     <div class="incomplete-notice">
       This section is incomplete. The Style Expert agent failed during analysis.
       Error: {error_message}
     </div>
   </div>
   ```
3. Show failure summary to user:
   ```
   | Expert          | Status | Note                          |
   |-----------------|--------|-------------------------------|
   | Style Expert    | FAILED | Timeout after 120s            |
   | Arch Expert     | OK     | 8 findings, 3 recommendations|
   | Visual QA       | OK     | 4 screenshots captured        |
   ```
4. Incomplete CSS:
   ```css
   .incomplete-notice { padding: 20px; border: 1px dashed rgba(239,159,39,0.4); border-radius: 8px; color: #EF9F27; font-size: 13px; text-align: center; }
   ```
