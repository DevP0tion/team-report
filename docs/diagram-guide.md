---
id: diagram-guide
phase: [3]
depends: [html-template]
load-when: "rendering architecture or UI diagrams in report"
created: 2026-04-01
---

# Architecture / UI Diagram Guide

Visualize architecture, UI layouts, and component trees using **HTML mocking** instead of ASCII art or code blocks.

## Rendering Rules

- Use colors, rounded corners, icons/emoji, and shadows for realistic UI feel
- Express layer/hierarchy with `z-index` + overlap or indentation
- Assign role-appropriate colors to component boxes (see Color Guide below)
- Use CSS border + pseudo-element or SVG `<line>` for arrows/connections
- Show tooltip on hover/focus; add `tabindex="0"` to any `.arch-box[data-tip]` so keyboard users can trigger it

## HTML Example

```html
<div class="arch-diagram">
  <div class="arch-label">SvelteKit /battle</div>

  <div class="arch-layer" style="--layer-color: #5DCAA5;">
    <span class="arch-layer-tag">Svelte HUD (z-index: 20)</span>
    <div class="arch-row">
      <div class="arch-box arch-box--sm" style="--box-bg: #1a2e24; --box-border: #5DCAA5;">
        <span class="arch-box-icon">&#10084;</span> HP Bar
      </div>
      <div class="arch-box arch-box--sm" style="--box-bg: #1a2e24; --box-border: #5DCAA5;">
        <span class="arch-box-icon">&#9201;</span> Timer
      </div>
    </div>
  </div>

  <div class="arch-layer" style="--layer-color: #85B7EB;">
    <span class="arch-layer-tag">PixiJS Canvas (z-index: 10)</span>
    <div class="arch-tree">
      <div class="arch-node">pixi-viewport
        <div class="arch-children">
          <div class="arch-node arch-node--leaf">Background tiles <code>cacheAsTexture</code></div>
          <div class="arch-node arch-node--leaf">Enemy Container <code>cullableChildren</code></div>
          <div class="arch-node arch-node--leaf">Player Sprite</div>
        </div>
      </div>
    </div>
  </div>
</div>
```

## Diagram CSS

```css
.arch-diagram { padding: 20px; border-radius: 12px; background: rgba(255, 255, 255, 0.02); border: 1px solid rgba(255, 255, 255, 0.06); display: flex; flex-direction: column; gap: 12px; }
.arch-label { font-family: 'Geist Mono', monospace; font-size: 14px; color: #e2e4ed; font-weight: 500; }
.arch-layer { border: 1px solid var(--layer-color); border-radius: 10px; padding: 14px; background: rgba(255,255,255,0.02); background: color-mix(in srgb, var(--layer-color) 5%, transparent); position: relative; }
.arch-layer-tag { position: absolute; top: -9px; left: 14px; background: #0a0b10; padding: 0 8px; font-size: 11px; font-family: 'Geist Mono', monospace; color: var(--layer-color); }
.arch-row { display: flex; gap: 8px; flex-wrap: wrap; }
.arch-box { padding: 8px 14px; border-radius: 8px; background: var(--box-bg); border: 1px solid var(--box-border); font-size: 12px; color: #e2e4ed; cursor: default; transition: transform 0.1s; }
.arch-box:hover { transform: translateY(-2px); }
.arch-box--sm { min-width: 70px; text-align: center; }
.arch-box--slot { min-width: 64px; text-align: center; }
.arch-box--slot small { color: #8b8fa3; font-size: 10px; }
.arch-box-icon { font-size: 14px; margin-right: 4px; }

/* Tree structure */
.arch-tree { padding-left: 8px; }
.arch-node { font-size: 12px; color: #b5d4f4; padding: 4px 0; font-family: 'Geist Mono', monospace; }
.arch-node code { font-size: 10px; color: #8b8fa3; background: rgba(255, 255, 255, 0.06); padding: 1px 5px; border-radius: 3px; margin-left: 6px; }
.arch-children { padding-left: 20px; border-left: 1px dashed rgba(133, 183, 235, 0.3); margin-left: 4px; }
.arch-node--leaf { color: #8b8fa3; }
.arch-node--leaf::before { content: '\251C\2500 '; color: rgba(133, 183, 235, 0.4); }
.arch-children .arch-node--leaf:last-child::before { content: '\2514\2500 '; }

/* Hover tooltip */
.arch-box[data-tip]:hover::after, .arch-box[data-tip]:focus::after { content: attr(data-tip); position: absolute; bottom: calc(100% + 6px); left: 50%; transform: translateX(-50%); padding: 4px 10px; font-size: 11px; color: #e2e4ed; background: #1a1b24; border: 1px solid rgba(255, 255, 255, 0.1); border-radius: 6px; white-space: nowrap; z-index: 10; pointer-events: none; }
```

## Color Guide (by layer purpose)

| Layer | border/tag color | box-bg | Example |
|-------|-----------------|--------|---------|
| HUD/UI overlay | `#5DCAA5` (teal) | `#1a2e24` | HP bar, timer, gold |
| Canvas/rendering | `#85B7EB` (blue) | `#0e1a2a` | Pixi, viewport, sprite |
| Input/slots | `#EF9F27` (amber) | `#2a1f0a` | Skill slots, hotkeys |
| Data/state | `#AFA9EC` (purple) | `#1a1a2e` | ECS World, Store |
| Server/API | `#F09595` (red) | `#2a1414` | Supabase, endpoints |
| Structure/routes | `#8b8fa3` (gray) | `#141418` | SvelteKit routes |
