---
id: css-reference
phase: [3]
depends: [html-template, decision-ui, diagram-guide]
load-when: "generating CSS for HTML report"
created: 2026-04-01
---

# CSS Class Reference

All CSS classes that MUST be included when generating a report:

## Layout

| Class | Purpose |
|-------|---------|
| `.report-layout` | Full layout (flex: sidebar + main) |
| `.sidebar` | Left sidebar (220px fixed) |
| `.sb-header > .sb-stats > .sb-stat` | Decision stat cards (`.accept`, `.defer`, `.reject`) |
| `.sb-progress-wrap` | Progress bar area (`.sb-progress-fill`) |
| `.sb-pages > .sb-page` | Page list (`.active`, `.done`, `.sb-dot`, `.sb-badge`) |
| `.main-area` | Main content area |
| `.top-bar` | Top bar — page title + counter (sticky) |
| `.page` | Page section (`data-page`, `data-title` required) |
| `.page.active` | Currently displayed page |

## Decision UI

| Class | Purpose |
|-------|---------|
| `.inline-rec` | Inline recommendation card (`data-rec="R{N}"` required) |
| `.inline-rec-head` | Rec header (`.inline-rec-id`, `.inline-rec-title`, `.inline-rec-pri`) |
| `.d-btn` | Decision button (`.chosen-accept`, `.chosen-defer`, `.chosen-reject`) |
| `.inline-rec-note` | Comment input (`.visible` toggle, `rows="3"`, Enter = newline) |
| `.decision-note-display` | Read-only note display with `white-space: pre-wrap` (Decision Summary page) |
| `.decision-summary` | Decision Summary page — aggregate |
| `.export-btn` | Export/Copy/Clear buttons |
| `.flash-save` | Save notification toast |

## Content

| Class | Purpose |
|-------|---------|
| `.report-header` | Report header (gradient bg) |
| `.exec-summary` | Executive Summary box |
| `.stats-grid > .stat-card` | Stats card grid (inner: `.stat-value`, `.stat-label`) |
| `.swatch-grid > .swatch` | Color swatch grid |
| `.comparison-table` | Comparison table (`.color-dot`, `.tag`) |
| `.side-by-side > .side-panel` | Left/right comparison panel |
| `.token-table` | Token suggestion table (`.new-tag`) |
| `.code-block` | Code block (`.comment`, `.var`, `.val`) |
| `.team-grid > .team-card` | Team composition cards (inner: `.team-role`, `.team-scope`) |
| `.incomplete-notice` | Failed agent section notice |

## Architecture Diagrams

| Class | Purpose |
|-------|---------|
| `.arch-diagram` | Diagram container |
| `.arch-layer` | Layer box (`--layer-color` required) |
| `.arch-layer-tag` | Layer label (top tag) |
| `.arch-row` | Horizontal component row |
| `.arch-box` | Component box (`--box-bg`, `--box-border`) |
| `.arch-tree > .arch-node` | Tree structure (`.arch-children`, `.arch-node--leaf`) |
