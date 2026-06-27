---
id: html-template
phase: [3]
depends: []
load-when: "generating HTML report in Phase 3"
created: 2026-04-01
---

# HTML Report Template

## Page Structure

Reports are paginated. Each page is a `<div class="page" data-page="N" data-title="Title">`.

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

Inline `.inline-rec` cards are placed **right after the analysis text that produced the recommendation** (not in a separate page). See `decision-ui.md` for card spec.

## Layout HTML (Sidebar + Top Bar + Main)

```html
<div class="report-layout">

  <!-- Left Sidebar: Decision progress + page navigation -->
  <aside class="sidebar" id="sidebar">
    <div class="sb-header">
      <div class="sb-title">Decisions</div>
      <div class="sb-stats" id="sbStats">
        <div class="sb-stat accept"><span class="sb-stat-num" id="statAccept">0</span>accept</div>
        <div class="sb-stat defer"><span class="sb-stat-num" id="statDefer">0</span>defer</div>
        <div class="sb-stat reject"><span class="sb-stat-num" id="statReject">0</span>reject</div>
      </div>
    </div>
    <div class="sb-progress-wrap">
      <div class="sb-progress-label">
        <span>Progress</span>
        <span id="sbProgressText">0 / 0</span>
      </div>
      <div class="sb-progress">
        <div class="sb-progress-fill" id="sbProgressFill"></div>
      </div>
    </div>
    <nav class="sb-pages" id="sbPages">
      <!-- JS dynamically generates page list -->
    </nav>
  </aside>

  <!-- Main Area -->
  <div class="main-area">
    <header class="top-bar">
      <span class="top-bar-title" id="topBarTitle">1. Analysis Team</span>
      <div class="top-bar-nav">
        <!-- pageSelect shown only when totalPages >= 10 (see initPageSelect) -->
        <select id="pageSelect" class="page-select" aria-label="Jump to page" onchange="goToPage(parseInt(this.value,10))"></select>
        <span class="top-bar-counter" id="topBarCounter">1 / 8</span>
      </div>
    </header>

    <!-- Page 1 — Analysis Team (uses .team-grid > .team-card; see css-reference.md) -->
    <section class="page" data-page="1" data-title="Analysis Team" role="main" aria-label="Analysis Team">
      <h1>Analysis Team</h1>
      <div class="team-grid">
        <div class="team-card">
          <h3>Code Analysis Expert</h3>
          <p class="team-role">Explore</p>
          <p class="team-scope">src/components/**</p>
        </div>
        <div class="team-card">
          <h3>Visual QA Expert</h3>
          <p class="team-role">general-purpose</p>
          <p class="team-scope">Playwright screenshots</p>
        </div>
      </div>
    </section>

    <!-- Page 2 — Executive Summary (uses .exec-summary, .stats-grid > .stat-card) -->
    <section class="page" data-page="2" data-title="Executive Summary" role="main" aria-label="Executive Summary">
      <h1>Executive Summary</h1>
      <div class="exec-summary">
        <p><strong>Key findings:</strong> 3 critical, 5 medium issues across 14 files.</p>
      </div>
      <div class="stats-grid">
        <div class="stat-card"><div class="stat-value">8</div><div class="stat-label">Findings</div></div>
        <div class="stat-card"><div class="stat-value">6</div><div class="stat-label">Recommendations</div></div>
      </div>
      <!-- inline .inline-rec cards go right after the evidence text that produced them -->
    </section>
    <!-- Page 3~N — one <section class="page"> per expert; then Gap Analysis, Visual Impression, etc. -->
    <!-- Every page MUST be <section class="page" data-page data-title role="main">. Each .inline-rec needs data-rec="R{N}". -->
  </div>

</div>
```

## Page Navigation + Sidebar JavaScript

```javascript
// State
let currentPage = 1;
const pages = document.querySelectorAll('.page');
const totalPages = pages.length;

// Page Navigation
function goToPage(n) {
  if (n < 1 || n > totalPages) return;
  currentPage = n;
  const page = pages[n - 1];

  pages.forEach(p => p.classList.remove('active'));
  page.classList.add('active');

  document.getElementById('topBarTitle').textContent =
    `${n}. ${page.dataset.title}`;
  document.getElementById('topBarCounter').textContent =
    `${n} / ${totalPages}`;

  document.querySelectorAll('.sb-page').forEach((item, i) => {
    item.classList.remove('active');
    if (i + 1 < n) item.classList.add('done');
    else item.classList.remove('done');
    if (i + 1 === n) {
      item.classList.add('active');
      item.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
    }
  });

  document.querySelector('.main-area').scrollTo({ top: 0, behavior: 'instant' });

  const sel = document.getElementById('pageSelect');
  if (sel) sel.value = n;
  page.setAttribute('tabindex', '-1');
  page.focus();
}

function prevPage() { goToPage(currentPage - 1); }
function nextPage() { goToPage(currentPage + 1); }

// Keyboard navigation
document.addEventListener('keydown', (e) => {
  if (['TEXTAREA', 'INPUT', 'SELECT'].includes(e.target.tagName) || e.target.contentEditable === 'true') return;
  if (e.key === 'ArrowLeft')  prevPage();
  if (e.key === 'ArrowRight') nextPage();
});

// Sidebar Init
function initSidebar() {
  const container = document.getElementById('sbPages');
  if (!container) return;
  pages.forEach((p, i) => {
    const pageNum = i + 1;
    const recCount = p.querySelectorAll('.inline-rec').length;
    const item = document.createElement('div');
    item.className = 'sb-page';
    item.addEventListener('click', () => goToPage(pageNum));
    item.innerHTML = `
      <div class="sb-dot"></div>
      <span class="sb-page-label">${pageNum}. ${p.dataset.title}</span>
      ${recCount > 0
        ? `<span class="sb-badge pending" data-page="${pageNum}">0/${recCount}</span>`
        : ''}
    `;
    container.appendChild(item);
  });
}

// Page dropdown — shown only for long reports (10+ pages)
function initPageSelect() {
  const sel = document.getElementById('pageSelect');
  if (!sel) return;
  if (totalPages < 10) { sel.style.display = 'none'; return; }
  pages.forEach((p, i) => {
    const opt = document.createElement('option');
    opt.value = i + 1;
    opt.textContent = `${i + 1}. ${p.dataset.title}`;
    sel.appendChild(opt);
  });
}

// Decision Progress Tracking
function updateSidebarProgress() {
  const state = loadState();
  const recs = document.querySelectorAll('.inline-rec');
  let accept = 0, defer = 0, reject = 0;

  recs.forEach(rec => {
    const id = rec.dataset.rec;
    const action = state[id]?.action;
    if (action === 'accept') accept++;
    if (action === 'defer')  defer++;
    if (action === 'reject') reject++;
  });

  document.getElementById('statAccept').textContent = accept;
  document.getElementById('statDefer').textContent = defer;
  document.getElementById('statReject').textContent = reject;

  const answered = accept + defer + reject;
  const total = recs.length;
  document.getElementById('sbProgressText').textContent = `${answered} / ${total}`;
  document.getElementById('sbProgressFill').style.width =
    total > 0 ? `${(answered / total) * 100}%` : '0%';

  pages.forEach((p, i) => {
    const pageRecs = p.querySelectorAll('.inline-rec');
    if (pageRecs.length === 0) return;
    let pageAnswered = 0;
    pageRecs.forEach(rec => {
      if (state[rec.dataset.rec]?.action) pageAnswered++;
    });
    const badge = document.querySelector(`.sb-badge[data-page="${i + 1}"]`);
    if (badge) {
      badge.textContent = `${pageAnswered}/${pageRecs.length}`;
      badge.className = pageAnswered === pageRecs.length
        ? 'sb-badge complete' : 'sb-badge pending';
    }
  });
}
```

## Layout CSS

```css
/* Report Layout: Sidebar + Main */
.report-layout { display: flex; height: 100vh; overflow: hidden; }

/* Sidebar */
.sidebar { width: 220px; min-width: 220px; background: var(--bg-sidebar, #0d0e16); border-right: 1px solid rgba(255, 255, 255, 0.06); display: flex; flex-direction: column; overflow: hidden; }
.sb-header { padding: 16px 14px 12px; border-bottom: 1px solid rgba(255, 255, 255, 0.06); }
.sb-title { font-size: 11px; font-family: 'Geist Mono', monospace; color: var(--text-muted, #5a5e72); text-transform: uppercase; letter-spacing: 0.5px; margin-bottom: 8px; }
.sb-stats { display: flex; gap: 6px; }
.sb-stat { flex: 1; padding: 6px 4px; border-radius: 6px; text-align: center; font-size: 10px; font-family: 'Geist Mono', monospace; }
.sb-stat.accept { background: rgba(93, 202, 165, 0.1); color: #5DCAA5; }
.sb-stat.defer  { background: rgba(239, 159, 39, 0.1); color: #EF9F27; }
.sb-stat.reject { background: rgba(226, 75, 74, 0.1);  color: #F09595; }
.sb-stat-num { display: block; font-size: 16px; font-weight: 500; }

/* Sidebar Progress */
.sb-progress-wrap { padding: 12px 14px; }
.sb-progress-label { display: flex; justify-content: space-between; font-size: 11px; color: var(--text-muted, #5a5e72); margin-bottom: 4px; }
.sb-progress { height: 3px; background: rgba(255, 255, 255, 0.06); border-radius: 2px; }
.sb-progress-fill { height: 100%; border-radius: 2px; background: linear-gradient(90deg, #5DCAA5, #85B7EB); transition: width 0.3s ease; }

/* Sidebar Page List */
.sb-pages { flex: 1; overflow-y: auto; padding: 4px 8px; scrollbar-width: thin; scrollbar-color: rgba(255,255,255,0.1) transparent; }
.sb-pages::-webkit-scrollbar { width: 4px; }
.sb-pages::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.1); border-radius: 2px; }
.sb-page { display: flex; align-items: center; gap: 8px; padding: 7px 8px; border-radius: 6px; cursor: pointer; font-size: 12px; color: var(--text-muted, #5a5e72); transition: all 0.12s; margin-bottom: 2px; }
.sb-page:hover { background: rgba(255, 255, 255, 0.04); }
.sb-page.active { background: rgba(133, 183, 235, 0.08); color: #85B7EB; }
.sb-page.done { color: #8b8fa3; }
.sb-dot { width: 6px; height: 6px; border-radius: 50%; background: rgba(255, 255, 255, 0.1); flex-shrink: 0; }
.sb-page.done .sb-dot   { background: #5DCAA5; }
.sb-page.active .sb-dot { background: #85B7EB; box-shadow: 0 0 0 2px rgba(133,183,235,0.2); }
.sb-page-label { flex: 1; min-width: 0; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
.sb-badge { margin-left: auto; font-size: 9px; padding: 1px 5px; border-radius: 4px; font-family: 'Geist Mono', monospace; flex-shrink: 0; }
.sb-badge.pending  { background: rgba(239,159,39,0.15); color: #EF9F27; }
.sb-badge.complete { background: rgba(93,202,165,0.15); color: #5DCAA5; }

/* Main Area */
.main-area { flex: 1; display: flex; flex-direction: column; min-width: 0; overflow-y: auto; }

/* Top Bar */
.top-bar { position: sticky; top: 0; z-index: 50; padding: 10px 24px; background: var(--bg-nav, #0d0e16); border-bottom: 1px solid rgba(255, 255, 255, 0.06); display: flex; align-items: center; justify-content: space-between; }
.top-bar-title { font-size: 14px; font-weight: 500; font-family: 'Geist Mono', monospace; color: var(--text-primary, #e2e4ed); }
.top-bar-counter { font-size: 12px; font-family: 'Geist Mono', monospace; color: var(--text-muted, #5a5e72); }

/* Page Display */
.page { display: none; padding: 24px; animation: pageFadeIn 0.25s ease-out; }
.page.active { display: block; }
@keyframes pageFadeIn { from { opacity: 0; transform: translateY(8px); } to { opacity: 1; transform: translateY(0); } }

/* Page dropdown (long reports) + a11y */
.top-bar-nav { display: flex; align-items: center; gap: 10px; }
.page-select { padding: 4px 8px; font-size: 12px; font-family: 'Geist Mono', monospace; color: var(--text-primary, #e2e4ed); background: rgba(255,255,255,0.06); border: 1px solid rgba(255,255,255,0.1); border-radius: 5px; }
.page:focus { outline: 2px solid #85B7EB; outline-offset: 2px; }

/* Responsive (mobile / tablet) */
@media (max-width: 768px) {
  .report-layout { flex-direction: column; height: auto; }
  .sidebar { width: 100%; min-width: 0; flex-direction: row; overflow-x: auto; border-right: none; border-bottom: 1px solid rgba(255,255,255,0.06); }
  .sb-header, .sb-progress-wrap { display: none; }
  .sb-pages { flex-direction: row; }
  .page { padding: 16px 12px; }
}
```

## HTML Style Rules

- Standalone HTML (no external dependencies except Google Fonts CDN)
- Dark theme (`--bg: #0a0b10` family)
- `Geist Mono` (headings) + `Inter` (body) fonts
- Color swatches use `.swatch-color` div
- Comparison tables use `.comparison-table` + `.color-dot` + `.tag` pattern
- Grid layout (`grid-template-columns: repeat(auto-fill, ...)`)
