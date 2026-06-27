---
id: feedback-system
phase: [6]
depends: [decision-ui]
load-when: "collecting or referencing feedback in Phase 6"
created: 2026-04-01
---

# Feedback Collection & Progressive Improvement

## Feedback Storage

All feedback accumulates in a **single file per project**:

```
{project-root}/docs/reports/feedback.yaml
```

## Feedback YAML Format

```yaml
# docs/reports/feedback.yaml
schema_version: 2

# Accumulated lessons from all reports
lessons:
  pagination:
    - lesson: "10+ pages need select dropdown instead of prev/next"
      first_seen: 2026-03-31
      last_applied: 2026-04-01
      status: active    # active | archived
  layout:
    - lesson: "comparison-table >5 columns needs overflow-x: auto"
      first_seen: 2026-03-31
      status: active
  decision_ui:
    - lesson: "inline-rec cards should be near evidence text"
      first_seen: 2026-04-01
      status: active

# Report history with decisions
report_history:
  - slug: team-report-improvement
    date: 2026-04-01
    path: docs/reports/team-report-improvement.html
    decisions:
      R1: { action: accept, note: "" }
      R2: { action: accept, note: "" }
      R3: { action: reject, note: "desktop only" }
    feedback:
      - type: layout
        detail: "Gap Analysis page too long — split into 2 pages"
        severity: medium

updated_at: 2026-04-01T00:00:00Z
```

**Field rules:** `schema_version` required (current: 2). `report_history[].path` is repo-root-relative. `last_applied` optional (omit if never applied). `status` defaults to `active`. Multi-line `note`: use a YAML block scalar (`note: |-`). Schema changes must be additive — keep old fields readable.

## Collection Method A: Playwright localStorage Capture

After report generation, if user completes decisions in browser:

```
1. Open report HTML via Playwright
2. Extract localStorage decisions via page.evaluate()
3. Merge into docs/reports/feedback.yaml (decisions section)
4. Preserve existing feedback and lessons
```

```javascript
// Playwright extraction
async (page) => {
  await page.goto('{report-url}');
  // wait for the report's init to populate storage (non-fatal on timeout)
  await page.waitForFunction(
    () => localStorage.getItem('core-{slug}-decisions') !== null,
    { timeout: 5000 }
  ).catch(() => null);
  const decisions = await page.evaluate(() => {
    try { return JSON.parse(localStorage.getItem('core-{slug}-decisions') || '{}'); }
    catch { return {}; }
  });
  return JSON.stringify(decisions, null, 2);
}
```

## Collection Method B: User Direct Feedback

When user provides feedback via conversation:

```
1. Check docs/reports/feedback.yaml (create if missing)
2. Add to feedback section under matching report_history entry
3. Generalize to lessons section if applicable
```

**Trigger examples:**
```
/team-report feedback {slug} — {message}
```

## Feedback Reference (During Report Generation)

**Phase 1 must perform:**

```
1. Glob for docs/reports/feedback.yaml
2. If exists, read and extract:
   a. All active lessons (cross-topic)
   b. Same-topic report history and decisions
3. Apply lessons to HTML generation
4. Reference previous decisions for context
```

**Priority:**
1. Same topic decisions — highest priority, apply all
2. lessons section — apply all active lessons regardless of topic
3. feedback section — reference similar topic layout/style/ux feedback

**Lesson deprecation:** Lessons with `last_applied` > 90 days ago and `status: active` should be flagged for review. Set `status: archived` if no longer relevant.

## gap-detector Integration (R10)

When bkit gap-detector output is available, parse the machine-readable JSON block:

```markdown
<!-- MACHINE_PARSEABLE_GAPS -->
{
  "gaps": [
    {
      "id": "G1",
      "type": "missing",
      "priority": "high",
      "item": "Error code standardization",
      "design_ref": "api-spec.md:42",
      "fix_suggestion": "Add to ErrorCode enum"
    }
  ]
}
<!-- END_MACHINE_PARSEABLE_GAPS -->
```

Convert each gap to an inline decision card:
- Gap ID `G{N}` maps to `data-rec="G{N}"`
- Include `design_ref` as evidence link
- User can Accept (fix it), Defer, or Reject (design is correct)
