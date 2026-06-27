---
id: decision-ui
phase: [4]
depends: [html-template]
load-when: "generating inline decision cards in Phase 4"
created: 2026-04-01
---

# Inline Decision UI

## Placement Principle

- Each `.inline-rec` is placed **right after the analysis text that produced the recommendation**
- Multiple `.inline-rec` cards can appear on one page
- Pages without recommendations (Team, Visual Impression) have no `.inline-rec`
- `data-rec="R{N}"` — N is unique across the entire report

## Inline Recommendation Card HTML

```html
<!-- After analysis text -->
<p>ButtonGroup 외 3개 컴포넌트에서 #hex 직접 사용 발견...</p>

<div class="inline-rec" data-rec="R{N}">
  <div class="inline-rec-head">
    <span class="inline-rec-id">R{N}</span>
    <span class="inline-rec-title">{Title}</span>
    <span class="inline-rec-pri pri-{high|med|low}">{priority}</span>
  </div>
  <p class="inline-rec-desc">{Description}</p>
  <div class="inline-rec-btns">
    <button class="d-btn" data-action="accept" onclick="setDecision('R{N}','accept')">Accept</button>
    <button class="d-btn" data-action="defer" onclick="setDecision('R{N}','defer')">Defer</button>
    <button class="d-btn" data-action="reject" onclick="setDecision('R{N}','reject')">Reject</button>
    <button class="d-btn d-btn-note" onclick="toggleNote('R{N}')">+ note</button>
  </div>
  <div class="inline-rec-note" id="note-R{N}">
    <textarea rows="3" placeholder="Comment... (Enter for newline)" aria-label="Note for R{N}" oninput="saveNote('R{N}', this.value)"></textarea>
  </div>
</div>

<p>Next analysis content continues...</p>
```

## Decision Summary Page (Last Page)

- Aggregate all decisions (same data as sidebar)
- Export JSON button (download decisions)
- Copy to Clipboard button (text copy)
- Clear All button (reset)
- Highlight unanswered items + link to jump to their page
- **Note display**: Use `<pre class="decision-note-display">` with `white-space: pre-wrap` to preserve line breaks when rendering saved notes inline (not in textarea)

## localStorage Key Format

`core-{report-slug}-decisions`

## JavaScript Functions

```javascript
// Storage
const STORAGE_KEY = 'core-{report-slug}-decisions';
function loadState() {
  try { return JSON.parse(localStorage.getItem(STORAGE_KEY) || '{}'); }
  catch (e) { console.warn('[decision-ui] localStorage read failed:', e.message); return {}; }
}
function saveState(state) {
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(state));
  } catch (e) {
    // file:// or private mode can block storage — warn instead of losing silently
    console.warn('[decision-ui] storage blocked (file:// or private mode?):', e.message);
    alert('Decisions cannot be saved here (serve over http:// to persist). They will be lost on reload.');
  }
  flashSave();
}

// Save notification animation
function flashSave() {
  const el = document.getElementById('flashSave');
  if (!el) return;
  el.classList.add('show');
  setTimeout(() => el.classList.remove('show'), 800);
}

// Decision Control
function setDecision(rec, action) {
  if (!['accept', 'defer', 'reject'].includes(action)) return;
  const state = loadState();
  // Toggle off on re-click
  if (state[rec]?.action === action) {
    delete state[rec].action;
  } else {
    state[rec] = { ...state[rec], action };
  }
  saveState(state);
  renderDecisionButtons(rec);
  updateSidebarProgress();
}

function saveNote(rec, value) {
  const state = loadState();
  state[rec] = { ...state[rec], note: value };
  saveState(state);
}

function toggleNote(rec) {
  const noteEl = document.getElementById(`note-${rec}`);
  if (noteEl) noteEl.classList.toggle('visible');
}

// Rendering
function renderDecisionButtons(rec) {
  const state = loadState();
  const action = state[rec]?.action;
  const card = document.querySelector(`.inline-rec[data-rec="${rec}"]`);
  if (!card) return;
  card.querySelectorAll('.d-btn[data-action]').forEach(btn => {
    btn.classList.remove('chosen-accept', 'chosen-defer', 'chosen-reject');
    if (btn.dataset.action === action) {
      btn.classList.add(`chosen-${action}`);
    }
  });
}

function renderAllDecisions() {
  document.querySelectorAll('.inline-rec').forEach(card => {
    renderDecisionButtons(card.dataset.rec);
    const state = loadState();
    const note = state[card.dataset.rec]?.note;
    if (note) {
      const ta = card.querySelector('textarea');
      if (ta) ta.value = note;
    }
  });
}

// Export (Decision Summary page)
function exportJSON() {
  const state = loadState();
  const blob = new Blob([JSON.stringify(state, null, 2)], { type: 'application/json' });
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = `${STORAGE_KEY}.json`;
  a.click();
  URL.revokeObjectURL(a.href);
}

function copyClipboard() {
  const state = loadState();
  const recs = document.querySelectorAll('.inline-rec');
  let text = '# Report Decisions\n\n';
  recs.forEach(r => {
    const id = r.dataset.rec;
    const title = r.querySelector('.inline-rec-title')?.textContent || '';
    const action = state[id]?.action || 'pending';
    const note = state[id]?.note || '';
    text += `- ${id}: ${title} → ${action}\n`;
    if (note) { note.split('\n').forEach(line => { text += `  > ${line}\n`; }); }
  });
  navigator.clipboard.writeText(text).then(() => flashSave()).catch(err => {
    console.error('Clipboard write failed:', err);
    // Fallback: select textarea
    const ta = document.createElement('textarea');
    ta.value = text;
    document.body.appendChild(ta);
    ta.select();
    document.execCommand('copy');
    document.body.removeChild(ta);
    flashSave();
  });
}

function clearAll() {
  if (confirm('Clear all decisions?')) {
    localStorage.removeItem(STORAGE_KEY);
    location.reload();
  }
}

// Init
document.addEventListener('DOMContentLoaded', () => {
  initSidebar();
  initPageSelect();
  goToPage(1);
  renderAllDecisions();
  updateSidebarProgress();
});
```

## Decision CSS

```css
/* Inline Recommendation Card */
.inline-rec { border: 1px solid rgba(239, 159, 39, 0.25); border-radius: 8px; padding: 12px 14px; margin: 16px 0; background: rgba(239, 159, 39, 0.03); }
.inline-rec-head { display: flex; align-items: center; gap: 8px; margin-bottom: 6px; }
.inline-rec-id { font-size: 11px; font-family: 'Geist Mono', monospace; color: #EF9F27; background: rgba(239, 159, 39, 0.15); padding: 2px 6px; border-radius: 4px; }
.inline-rec-title { font-size: 13px; font-weight: 500; color: var(--text-primary, #e2e4ed); }
.inline-rec-pri { font-size: 10px; padding: 2px 6px; border-radius: 4px; margin-left: auto; }
.pri-high { background: rgba(226,75,74,0.15); color: #F09595; }
.pri-med  { background: rgba(239,159,39,0.15); color: #EF9F27; }
.pri-low  { background: rgba(255,255,255,0.08); color: #8b8fa3; }
.inline-rec-desc { font-size: 12px; color: var(--text-secondary, #8b8fa3); margin-bottom: 10px; line-height: 1.5; }

/* Decision Buttons */
.inline-rec-btns { display: flex; gap: 6px; }
.d-btn { padding: 5px 12px; font-size: 11px; border-radius: 5px; cursor: pointer; border: 1px solid rgba(255, 255, 255, 0.08); background: rgba(255, 255, 255, 0.04); color: #8b8fa3; transition: all 0.12s; }
.d-btn:hover { background: rgba(255, 255, 255, 0.08); }
.d-btn.chosen-accept { background: rgba(93, 202, 165, 0.15); color: #5DCAA5; border-color: rgba(93, 202, 165, 0.3); }
.d-btn.chosen-defer { background: rgba(239, 159, 39, 0.15); color: #EF9F27; border-color: rgba(239, 159, 39, 0.3); }
.d-btn.chosen-reject { background: rgba(226, 75, 74, 0.15); color: #F09595; border-color: rgba(226, 75, 74, 0.3); }
.d-btn-note { margin-left: auto; font-size: 10px; }

/* Note Input */
.inline-rec-note { display: none; margin-top: 8px; }
.inline-rec-note.visible { display: block; }
.inline-rec-note textarea { width: 100%; background: rgba(255, 255, 255, 0.04); border: 1px solid rgba(255, 255, 255, 0.08); border-radius: 5px; padding: 8px 10px; font-size: 11px; color: var(--text-primary, #e2e4ed); resize: vertical; min-height: 54px; font-family: inherit; line-height: 1.5; white-space: pre-wrap; }
.decision-note-display { white-space: pre-wrap; word-break: break-word; background: rgba(255, 255, 255, 0.04); border: 1px solid rgba(255, 255, 255, 0.06); border-radius: 5px; padding: 6px 10px; font-size: 11px; color: var(--text-secondary, #8b8fa3); line-height: 1.5; margin: 4px 0 0; font-family: inherit; }

/* Flash save notification */
.flash-save { position: fixed; bottom: 20px; right: 20px; background: rgba(93,202,165,.15); color: #5DCAA5; padding: 8px 16px; border-radius: 6px; font-size: 12px; font-family: 'Geist Mono', monospace; opacity: 0; transition: opacity .3s; pointer-events: none; z-index: 100; }
.flash-save.show { opacity: 1; }
```

**Required HTML element for flash notification:**
```html
<div class="flash-save" id="flashSave">Saved</div>
```
