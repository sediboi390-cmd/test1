# Shot Builder Studio — Phase 1 Implementation Plan

> **For agentic workers:** This plan is implemented manually in a single session. Each task is small and self-contained. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a fully working faceless-content storyboard builder at `storyboard/index.html` with Card mode, Templates, Timeline, Character Manager, Save/Load, and HTML/PDF export.

**Architecture:** Single self-contained HTML file. Pure HTML/CSS/JS. State lives in a single `App` object held in JS, persisted to LocalStorage. Render functions read state and rebuild the DOM. SVG panel scenes are generated from panel field values via a simple builder.

**Tech Stack:** Pure HTML/CSS/JS, SVG, Canvas API, LocalStorage, JSZip (Phase 2). No build tools, no framework.

**Spec:** `docs/superpowers/specs/2026-05-27-shot-builder-studio-design.md`

---

## File Structure

Phase 1 ships as a single file:

- **Create:** `storyboard/index.html` — full app (HTML, CSS, JS inline)
- **Backup:** existing `storyboard/index.html` → renamed `storyboard/shadow-strike.html` (preserves the original fight-scene storyboard for reference)
- **Keep:** `storyboard/preview.html` (UI mockup, not modified)

Single-file rationale: the spec calls for it, deployment is just copying one file, and the app is small enough that splitting would add friction without value.

---

## Tasks

### Task 1: Preserve existing storyboard and create app skeleton


**Files:**
- Rename: `storyboard/index.html` → `storyboard/shadow-strike.html`
- Create: `storyboard/index.html` (new app skeleton)

- [ ] **Step 1: Backup the existing storyboard**
  ```bash
  git mv storyboard/index.html storyboard/shadow-strike.html
  ```

- [ ] **Step 2: Create the new index.html with HTML skeleton, CSS, and empty `<script>` block**
  Use the `preview.html` file as the visual reference. Copy the calm-minimalist palette CSS, header, sidebar, main canvas, status bar, and editor drawer markup. Replace the static empty-state with an `<div id="app"></div>` mount point that the JS will fill. Add an empty `<script>` tag at the bottom of `<body>`.

- [ ] **Step 3: Verify it loads in a browser**
  Open `storyboard/index.html` directly in a browser. You should see the calm-styled empty layout with sidebar, header, and the empty main area. No JS yet, so nothing interactive.

- [ ] **Step 4: Commit**
  ```bash
  git add storyboard/index.html storyboard/shadow-strike.html
  git commit -m "feat(storyboard): create Shot Builder Studio app skeleton"
  ```

---

### Task 2: State management and LocalStorage persistence

**Files:**
- Modify: `storyboard/index.html` (JS section)

- [ ] **Step 1: Define the global App state object**
  Inside the `<script>` block, add the project state model:
  ```js
  const STORAGE_KEY = 'shotBuilderStudio.project.v1';

  const App = {
    project: {
      name: 'Untitled',
      aspectRatio: '16:9',
      characters: [],   // [{ id, name, role, color }]
      panels: [],       // see panel data model in spec §10
      acts: ['Act I'],  // simple array of act names; panel.act = index
    },
    ui: {
      view: 'grid',           // 'grid' | 'outline'
      activeMode: 'card',     // 'card' | 'template' | 'text' | 'sketch'
      selectedPanelId: null,
    },
  };
  ```

- [ ] **Step 2: Add load + save helpers**
  ```js
  function saveToStorage() {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(App.project));
  }

  function loadFromStorage() {
    const raw = localStorage.getItem(STORAGE_KEY);
    if (!raw) return false;
    try {
      App.project = JSON.parse(raw);
      return true;
    } catch (e) {
      console.warn('Could not parse saved project:', e);
      return false;
    }
  }
  ```

- [ ] **Step 3: Add auto-save (every 30s) and a manual save trigger**
  ```js
  setInterval(saveToStorage, 30000);
  window.addEventListener('keydown', (e) => {
    if ((e.ctrlKey || e.metaKey) && e.key === 's') {
      e.preventDefault();
      saveToStorage();
      flashStatus('Saved');
    }
  });
  ```

- [ ] **Step 4: Add a tiny `uid()` helper for panel/character IDs**
  ```js
  function uid() {
    return Date.now().toString(36) + Math.random().toString(36).slice(2, 8);
  }
  ```

- [ ] **Step 5: Manually test in browser DevTools**
  Open the page, run in console:
  ```js
  App.project.name = 'Test'; saveToStorage();
  // Reload page, then:
  loadFromStorage(); console.log(App.project.name);
  // Expected: "Test"
  ```

- [ ] **Step 6: Commit**
  ```bash
  git add storyboard/index.html
  git commit -m "feat(storyboard): add state management and LocalStorage persistence"
  ```

---

### Task 3: Render loop and empty state


**Files:**
- Modify: `storyboard/index.html`

- [ ] **Step 1: Add a top-level `render()` function**
  ```js
  function render() {
    renderSidebar();
    renderTimeline();
    renderViewToggle();
    renderMain();
    renderStatusBar();
  }
  ```

- [ ] **Step 2: Implement `renderMain()` with empty-state branch**
  ```js
  function renderMain() {
    const main = document.getElementById('panel-grid');
    if (App.project.panels.length === 0) {
      main.innerHTML = `
        <div class="empty-state">
          <div class="empty-icon">◇</div>
          <div class="empty-title">Your storyboard is empty</div>
          <div class="empty-desc">
            Add your first panel to begin, or load the sample storyboard.
          </div>
          <div class="empty-actions">
            <button class="btn btn-primary" id="btn-add-first">+ Add First Panel</button>
            <button class="btn btn-ghost" id="btn-load-sample-empty">📖 Load Sample</button>
          </div>
        </div>`;
      document.getElementById('btn-add-first').onclick = () => addPanel();
      document.getElementById('btn-load-sample-empty').onclick = () => loadSample();
      return;
    }
    // panels exist — see Task 6
    main.innerHTML = renderPanelGrid();
  }
  ```

- [ ] **Step 3: Stub the other render functions**
  ```js
  function renderSidebar() { /* Task 4 */ }
  function renderTimeline() { /* Task 7 */ }
  function renderViewToggle() { /* Task 9 */ }
  function renderStatusBar() { /* Task 5 */ }
  function renderPanelGrid() { return ''; /* Task 6 */ }
  function addPanel() { /* Task 6 */ }
  function loadSample() { /* Task 12 */ }
  function flashStatus(msg) { /* Task 5 */ }
  ```

- [ ] **Step 4: Boot on page load**
  ```js
  document.addEventListener('DOMContentLoaded', () => {
    if (loadFromStorage()) {
      // existing project loaded
    } else {
      // fresh project (App.project already initialised)
    }
    render();
  });
  ```

- [ ] **Step 5: Verify in browser**
  Reload the page. You should see the empty-state card centered in the main area. Buttons exist but do nothing yet.

- [ ] **Step 6: Commit**
  ```bash
  git add storyboard/index.html
  git commit -m "feat(storyboard): add render loop and empty state"
  ```

---

### Task 4: Sidebar — modes, characters, exports, project info

**Files:**
- Modify: `storyboard/index.html`

- [ ] **Step 1: Implement `renderSidebar()`**
  ```js
  function renderSidebar() {
    const modes = [
      { id: 'card', icon: '📋', label: 'Card' },
      { id: 'template', icon: '🎬', label: 'Templates' },
      { id: 'text', icon: '✨', label: 'Text to Panel' },
      { id: 'sketch', icon: '✏️', label: 'Sketch' },
    ];
    const modesHtml = modes.map(m => `
      <div class="mode-tab ${App.ui.activeMode === m.id ? 'active' : ''}" data-mode="${m.id}">
        <span class="mode-icon">${m.icon}</span> ${m.label}
      </div>`).join('');

    const charsHtml = App.project.characters.length === 0
      ? `<div style="color:var(--muted);font-size:0.7rem;margin-bottom:8px">No characters yet</div>`
      : App.project.characters.map(c => `
          <span class="char-chip"><span class="char-dot" style="background:${c.color}"></span>${escapeHtml(c.name)}</span>`).join('');

    document.getElementById('sidebar').innerHTML = `
      <div class="sidebar-section">
        <div class="sidebar-label">Mode</div>
        ${modesHtml}
        <div class="sample-divider"></div>
        <div class="sample-tab" id="btn-load-sample">
          <span class="mode-icon">📖</span><span>Load Sample</span>
        </div>
        <div class="sample-hint">See a pre-filled example storyboard with all 4 modes in action.</div>
      </div>

      <div class="sidebar-section">
        <div class="sidebar-label">Characters</div>
        <div>${charsHtml}</div>
        <button class="btn btn-ghost" id="btn-add-character" style="margin-top:10px;width:100%;font-size:0.65rem">+ Add Character</button>
      </div>

      <div class="sidebar-section">
        <div class="sidebar-label">Export</div>
        <div class="export-item" data-export="html"><span class="export-icon">🌐</span> Shareable HTML</div>
        <div class="export-item" data-export="pdf"><span class="export-icon">📄</span> PDF</div>
        <div class="export-item" data-export="json"><span class="export-icon">💾</span> JSON Backup</div>
      </div>

      <div class="sidebar-section">
        <div class="sidebar-label">Project Info</div>
        <div class="project-stat">Panels <span>${App.project.panels.length}</span></div>
        <div class="project-stat">Duration <span>${formatDuration(totalDuration())}</span></div>
        <div class="project-stat">Acts <span>${App.project.acts.length}</span></div>
        <div class="project-stat">Format
          <select class="ratio-select" id="ratio-select">
            <option value="16:9" ${App.project.aspectRatio === '16:9' ? 'selected' : ''}>16:9 — YouTube</option>
            <option value="9:16" ${App.project.aspectRatio === '9:16' ? 'selected' : ''}>9:16 — TikTok / Shorts</option>
            <option value="1:1"  ${App.project.aspectRatio === '1:1'  ? 'selected' : ''}>1:1 — Instagram</option>
          </select>
        </div>
      </div>
    `;

    // Wire up listeners
    document.querySelectorAll('.mode-tab').forEach(el => {
      el.onclick = () => { App.ui.activeMode = el.dataset.mode; render(); };
    });
    document.getElementById('btn-load-sample').onclick = () => loadSample();
    document.getElementById('btn-add-character').onclick = () => addCharacter();
    document.querySelectorAll('.export-item').forEach(el => {
      el.onclick = () => exportProject(el.dataset.export);
    });
    document.getElementById('ratio-select').onchange = (e) => {
      App.project.aspectRatio = e.target.value;
      saveToStorage();
      render();
    };
  }
  ```

- [ ] **Step 2: Add helper functions**
  ```js
  function totalDuration() {
    return App.project.panels.reduce((sum, p) => sum + (p.duration || 0), 0);
  }
  function formatDuration(s) {
    if (s < 60) return `${s}s`;
    return `${Math.floor(s / 60)}m ${s % 60}s`;
  }
  function escapeHtml(s) {
    return String(s).replace(/[&<>"']/g, c => ({
      '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;'
    }[c]));
  }
  ```

- [ ] **Step 3: Stub `addCharacter()` and `exportProject()`**
  ```js
  function addCharacter() { /* Task 8 */ }
  function exportProject(kind) { /* Task 11 */ }
  ```

- [ ] **Step 4: Verify**
  Reload page. Sidebar shows all four modes (Card highlighted), Load Sample, empty Characters section, three export options, and Project Info with Format dropdown. Clicking modes switches the highlight. Changing the Format dropdown updates state (verify via DevTools `App.project.aspectRatio`).

- [ ] **Step 5: Commit**
  ```bash
  git add storyboard/index.html
  git commit -m "feat(storyboard): render sidebar with modes, characters, exports, project info"
  ```

---

### Task 5: Status bar


**Files:**
- Modify: `storyboard/index.html`

- [ ] **Step 1: Implement `renderStatusBar()` and `flashStatus()`**
  ```js
  function renderStatusBar() {
    const total = App.project.panels.length;
    const dur = formatDuration(totalDuration());
    const acts = App.project.acts.length;
    const ratio = App.project.aspectRatio;
    const words = totalVoiceoverWords();

    document.getElementById('status-bar').innerHTML = `
      <span><b>${total}</b> panels</span>
      <span><b>${dur}</b> total</span>
      <span><b>${acts}</b> acts</span>
      <span><b>${ratio}</b></span>
      <span><b>${words}</b> words voiceover</span>
      <span class="status-bar-right" id="status-msg">Shot Builder Studio</span>
    `;
  }

  function totalVoiceoverWords() {
    return App.project.panels.reduce((sum, p) => {
      const w = (p.voiceover || '').trim();
      return sum + (w ? w.split(/\s+/).length : 0);
    }, 0);
  }

  function flashStatus(msg) {
    const el = document.getElementById('status-msg');
    if (!el) return;
    const original = el.textContent;
    el.textContent = msg;
    el.style.color = 'var(--sage)';
    setTimeout(() => { el.textContent = original; el.style.color = ''; }, 1500);
  }
  ```

- [ ] **Step 2: Verify**
  Reload page. Status bar shows `0 panels · 0s total · 1 acts · 16:9 · 0 words voiceover`. Press Ctrl+S → message briefly says "Saved" in sage green.

- [ ] **Step 3: Commit**
  ```bash
  git add storyboard/index.html
  git commit -m "feat(storyboard): add status bar with totals and save flash"
  ```

---

### Task 6: Card mode — add panel and basic panel card rendering

**Files:**
- Modify: `storyboard/index.html`

- [ ] **Step 1: Implement `addPanel()`**
  ```js
  function addPanel() {
    const newPanel = {
      id: uid(),
      act: App.project.panels.length === 0 ? 0 : App.project.panels[App.project.panels.length - 1].act,
      order: App.project.panels.length,
      creationMode: App.ui.activeMode,
      label: 'Untitled Panel',
      shotType: 'medium',
      cameraAngle: 'eye',
      cameraMove: 'static',
      duration: 5,
      background: 'studio',
      lighting: 'bright',
      emotion: 'neutral',
      characters: [],
      visualStyle: 'animated-realism',
      videoStyle: 'youtube',
      description: '',
      voiceover: '',
      motionNotes: '',
      audioNotes: '',
      sketchDataUrl: null,
      templateId: null,
      textPrompt: null,
    };
    App.project.panels.push(newPanel);
    App.ui.selectedPanelId = newPanel.id;
    saveToStorage();
    render();
  }
  ```

- [ ] **Step 2: Implement `renderPanelGrid()`**
  ```js
  function renderPanelGrid() {
    // Group panels by act
    const grouped = App.project.acts.map((name, idx) => ({
      name,
      idx,
      panels: App.project.panels.filter(p => p.act === idx),
    }));

    return grouped.map(g => `
      <div class="act-heading">${escapeHtml(g.name)}</div>
      <div class="grid">
        ${g.panels.map(renderPanelCard).join('')}
        <div class="add-card" data-act="${g.idx}">
          <div class="add-card-icon">＋</div>
          <div class="add-card-label">Add Panel</div>
        </div>
      </div>
    `).join('');
  }

  function renderPanelCard(p) {
    const idx = App.project.panels.indexOf(p) + 1;
    const num = String(idx).padStart(2, '0');
    const selected = App.ui.selectedPanelId === p.id ? 'selected' : '';
    return `
      <div class="panel-card ${selected}" data-panel-id="${p.id}">
        <span class="panel-badge">${num}</span>
        <span class="panel-dur">${p.duration}s</span>
        <span class="panel-mode m-${p.creationMode}">${capitalize(p.creationMode)}</span>
        <div class="panel-scene">${renderPanelScene(p)}</div>
        <div class="panel-body">
          <div class="panel-title">${escapeHtml(p.label)}</div>
          <div class="tags">
            <span class="tag t-shot">${escapeHtml(p.shotType)}</span>
            <span class="tag t-move">${escapeHtml(p.cameraMove)}</span>
            <span class="tag t-emotion">${escapeHtml(p.emotion)}</span>
            <span class="tag t-light">${escapeHtml(p.lighting)}</span>
          </div>
          <div class="panel-desc">${escapeHtml(p.description) || '<em style="color:var(--muted)">No description yet</em>'}</div>
          <div class="panel-actions">
            <button class="panel-btn" data-action="edit" data-id="${p.id}">Edit</button>
            <button class="panel-btn" data-action="dupe" data-id="${p.id}">Duplicate</button>
            <button class="panel-btn del" data-action="delete" data-id="${p.id}">Delete</button>
          </div>
        </div>
      </div>
    `;
  }

  function capitalize(s) { return s.charAt(0).toUpperCase() + s.slice(1); }
  ```

- [ ] **Step 3: Stub `renderPanelScene()` for now**
  ```js
  function renderPanelScene(p) {
    // Minimal placeholder; real generator in Task 10
    return `<svg viewBox="0 0 400 160"><rect width="400" height="160" fill="#EDECEA"/><text x="200" y="85" text-anchor="middle" fill="var(--muted)" font-size="11" font-family="Inter">${escapeHtml(p.label)}</text></svg>`;
  }
  ```

- [ ] **Step 4: Wire up panel card clicks (delegated)**
  Inside `renderMain()`, after setting `main.innerHTML`, attach delegated listeners:
  ```js
  main.onclick = (e) => {
    const action = e.target.dataset.action;
    const id = e.target.dataset.id;
    if (action === 'edit') { App.ui.selectedPanelId = id; render(); }
    if (action === 'dupe') { duplicatePanel(id); }
    if (action === 'delete') { deletePanel(id); }
    if (e.target.classList.contains('add-card')) {
      addPanel();
    }
  };

  function duplicatePanel(id) {
    const p = App.project.panels.find(x => x.id === id);
    if (!p) return;
    const copy = { ...p, id: uid(), order: App.project.panels.length, label: p.label + ' (copy)' };
    App.project.panels.push(copy);
    saveToStorage();
    render();
  }
  function deletePanel(id) {
    if (!confirm('Delete this panel?')) return;
    App.project.panels = App.project.panels.filter(p => p.id !== id);
    if (App.ui.selectedPanelId === id) App.ui.selectedPanelId = null;
    saveToStorage();
    render();
  }
  ```

- [ ] **Step 5: Verify**
  Click "+ Add First Panel" → a panel card appears with "Untitled Panel". Click "Add Panel" tile → second panel appears. Duplicate works. Delete (with confirmation) works.

- [ ] **Step 6: Commit**
  ```bash
  git add storyboard/index.html
  git commit -m "feat(storyboard): add panel CRUD and card grid rendering"
  ```

---

### Task 7: Timeline bar


**Files:**
- Modify: `storyboard/index.html`

- [ ] **Step 1: Implement `renderTimeline()`**
  ```js
  function renderTimeline() {
    const bar = document.getElementById('timeline-bar');
    if (App.project.panels.length === 0) {
      bar.innerHTML = `<span class="timeline-bar-empty">Timeline · empty</span>`;
      return;
    }
    const actColors = ['var(--teal)', 'var(--sage)', 'var(--lavender)', 'var(--clay)'];
    const html = App.project.acts.map((name, actIdx) => {
      const panels = App.project.panels.filter(p => p.act === actIdx);
      const dot = `<div class="tl-dot" style="background:${actColors[actIdx % 4]}"></div>`;
      const pill = `<span class="tl-act-pill tl-act-${(actIdx % 2) + 1}">${escapeHtml(name)}</span>`;
      const thumbs = panels.map(p => `
        <div class="tl-line"></div>
        <div class="tl-thumb ${App.ui.selectedPanelId === p.id ? 'active' : ''}" data-panel-id="${p.id}">
          ${renderPanelScene(p)}
        </div>`).join('');
      return `${pill}${dot}${thumbs}<div class="tl-line"></div><div class="tl-add" data-act="${actIdx}">+</div><div style="width:16px"></div>`;
    }).join('');
    bar.innerHTML = html;

    bar.onclick = (e) => {
      const thumb = e.target.closest('.tl-thumb');
      if (thumb) { App.ui.selectedPanelId = thumb.dataset.panelId; render(); return; }
      if (e.target.classList.contains('tl-add')) { addPanel(); }
    };
  }
  ```

- [ ] **Step 2: Verify**
  With panels added, the timeline bar shows the act pill, dot, mini thumbnails, and a `+` at the end. Clicking a thumb selects that panel (highlights both the thumb and the panel card).

- [ ] **Step 3: Commit**
  ```bash
  git add storyboard/index.html
  git commit -m "feat(storyboard): add timeline bar with mini thumbnails"
  ```

---

### Task 8: Character Manager

**Files:**
- Modify: `storyboard/index.html`

- [ ] **Step 1: Implement `addCharacter()`**
  ```js
  const CHARACTER_COLORS = [
    '#4A8C8C', '#6B8F71', '#A0714F', '#7B7BA8', '#C0524A', '#9A9590',
  ];

  function addCharacter() {
    if (App.project.characters.length >= 6) {
      alert('Maximum of 6 characters per project.');
      return;
    }
    const name = prompt('Character name:', 'Hero');
    if (!name) return;
    const role = prompt('Role (hero / villain / narrator / host / product / extra):', 'hero');
    if (!role) return;
    const usedColors = new Set(App.project.characters.map(c => c.color));
    const color = CHARACTER_COLORS.find(c => !usedColors.has(c)) || '#9A9590';
    App.project.characters.push({ id: uid(), name, role, color });
    saveToStorage();
    render();
  }
  ```

- [ ] **Step 2: Verify**
  Click "+ Add Character" in sidebar. Fill in two prompts. New character chip appears with a colour dot. Add up to 6, then the alert blocks further adds.

- [ ] **Step 3: Commit**
  ```bash
  git add storyboard/index.html
  git commit -m "feat(storyboard): add character manager with colour assignment"
  ```

---

### Task 9: View toggle (Grid / Outline) and voiceover summary

**Files:**
- Modify: `storyboard/index.html`

- [ ] **Step 1: Implement `renderViewToggle()`**
  ```js
  function renderViewToggle() {
    const row = document.getElementById('view-toggle-row');
    const totalWords = totalVoiceoverWords();
    const totalRead = Math.ceil(totalWords / 150 * 60); // 150 wpm → seconds
    row.innerHTML = `
      <div class="view-toggle">
        <button class="view-tab ${App.ui.view === 'grid' ? 'active' : ''}" data-view="grid">
          <span class="view-tab-icon">▦</span> Grid
        </button>
        <button class="view-tab ${App.ui.view === 'outline' ? 'active' : ''}" data-view="outline">
          <span class="view-tab-icon">☰</span> Outline
        </button>
      </div>
      <div class="view-meta">
        Voiceover total: <strong>${totalWords} words</strong> · ~<strong>${totalRead}s</strong> read time
      </div>
    `;
    row.querySelectorAll('.view-tab').forEach(el => {
      el.onclick = () => { App.ui.view = el.dataset.view; render(); };
    });
  }
  ```

- [ ] **Step 2: Update `renderMain()` to branch on view**
  ```js
  function renderMain() {
    const main = document.getElementById('panel-grid');
    if (App.project.panels.length === 0) {
      main.innerHTML = renderEmptyState();
      wireEmptyState(main);
      return;
    }
    main.innerHTML = App.ui.view === 'outline' ? renderOutlineView() : renderPanelGrid();
    wirePanelActions(main);
  }
  ```
  Extract the empty-state HTML and listeners into helpers `renderEmptyState()` and `wireEmptyState(main)`. Extract the click delegation from Task 6 step 4 into `wirePanelActions(main)`.

- [ ] **Step 3: Implement `renderOutlineView()`**
  ```js
  function renderOutlineView() {
    const rows = App.project.panels.map((p, i) => {
      const num = String(i + 1).padStart(2, '0');
      const wordCount = (p.voiceover || '').trim().split(/\s+/).filter(Boolean).length;
      const vo = p.voiceover ? `"${escapeHtml(p.voiceover.slice(0, 100))}${p.voiceover.length > 100 ? '…' : ''}"` : '<em style="color:var(--muted)">No voiceover yet</em>';
      return `
        <div class="outline-row" data-panel-id="${p.id}">
          <div class="outline-num">${num}</div>
          <div class="outline-body">
            <div class="outline-title">${escapeHtml(p.label)} <span class="outline-dur">${p.duration}s</span></div>
            <div class="outline-vo">${vo}</div>
          </div>
          <div class="outline-meta">${wordCount} words</div>
        </div>`;
    }).join('');
    return `<div class="outline-list">${rows}<button class="btn btn-ghost" id="btn-add-outline" style="margin:12px 0">+ Add Panel</button></div>`;
  }
  ```

- [ ] **Step 4: Add CSS for outline view**
  In the `<style>` block, add:
  ```css
  .outline-list { background: var(--surface); border: 1px solid var(--border); border-radius: 10px; padding: 8px; }
  .outline-row { display: flex; gap: 14px; padding: 12px 14px; border-radius: 7px; cursor: pointer; transition: background 0.13s; }
  .outline-row:hover { background: var(--bg); }
  .outline-num { font-family: 'DM Sans', sans-serif; font-size: 0.85rem; font-weight: 600; color: var(--muted); width: 28px; }
  .outline-body { flex: 1; }
  .outline-title { font-family: 'DM Sans', sans-serif; font-size: 0.85rem; font-weight: 600; color: var(--text); margin-bottom: 4px; }
  .outline-dur { font-family: 'Inter', sans-serif; font-size: 0.65rem; font-weight: 400; color: var(--muted); margin-left: 6px; }
  .outline-vo { font-size: 0.7rem; color: var(--muted); line-height: 1.5; }
  .outline-meta { font-size: 0.65rem; color: var(--muted); align-self: center; }
  ```

- [ ] **Step 5: Verify**
  Add 2-3 panels. Click Outline → panel grid switches to a clean text list. Click Grid → returns to cards. The voiceover summary shows "0 words · ~0s" (since no voiceover entered yet).

- [ ] **Step 6: Commit**
  ```bash
  git add storyboard/index.html
  git commit -m "feat(storyboard): add Grid/Outline view toggle and voiceover summary"
  ```

---

### Task 10: SVG panel scene generator


**Files:**
- Modify: `storyboard/index.html`

- [ ] **Step 1: Replace the placeholder `renderPanelScene()` with the real generator**
  ```js
  const SCENE_BG_BY_MODE = {
    card:    '#EDECEA',
    template:'#EAE9F0',
    text:    '#E8F0E9',
    sketch:  '#F5EFE9',
  };
  const BG_TINT_BY_LIGHTING = {
    dark:       '#D9D5CF',
    bright:     '#FFFFFF',
    neon:       '#DFF0EF',
    sunset:     '#F5E2D0',
    spotlight:  '#FFFDE7',
    silhouette: '#C9C5BF',
  };
  function aspectFor(ratio) {
    if (ratio === '9:16') return { w: 90, h: 160 };
    if (ratio === '1:1')  return { w: 160, h: 160 };
    return { w: 400, h: 160 };
  }

  function renderPanelScene(p) {
    const { w, h } = aspectFor(App.project.aspectRatio);
    const baseBg = SCENE_BG_BY_MODE[p.creationMode] || '#EDECEA';
    const tint = BG_TINT_BY_LIGHTING[p.lighting] || baseBg;
    // Place 1-2 character silhouettes based on p.characters
    const chars = (p.characters || []).map(id => App.project.characters.find(c => c.id === id)).filter(Boolean);
    const silhouettes = chars.slice(0, 2).map((c, i) => {
      const cx = chars.length === 1 ? w / 2 : (i === 0 ? w * 0.3 : w * 0.7);
      const cy = h * 0.55;
      return `
        <ellipse cx="${cx}" cy="${cy - 18}" rx="11" ry="12" fill="#FFFFFF" stroke="${c.color}" stroke-width="1.2"/>
        <rect x="${cx - 10}" y="${cy - 6}" width="20" height="38" rx="4" fill="#FFFFFF" stroke="${c.color}" stroke-width="1"/>
        <line x1="${cx - 10}" y1="${cy + 2}" x2="${cx - 24}" y2="${cy + 22}" stroke="${c.color}" stroke-width="1.5"/>
        <line x1="${cx + 10}" y1="${cy + 2}" x2="${cx + 24}" y2="${cy + 22}" stroke="${c.color}" stroke-width="1.5"/>
      `;
    }).join('');
    return `
      <svg viewBox="0 0 ${w} ${h}" preserveAspectRatio="xMidYMid meet" xmlns="http://www.w3.org/2000/svg">
        <rect width="${w}" height="${h}" fill="${tint}"/>
        <rect x="0" y="${h - 20}" width="${w}" height="20" fill="rgba(0,0,0,0.04)"/>
        ${silhouettes}
      </svg>`;
  }
  ```

- [ ] **Step 2: Verify**
  Add a panel. Add a character. Open the editor (Task 11) and assign the character to the panel. The panel scene should now show a stick-figure silhouette with that character's stroke colour. Switching aspect ratio in sidebar reshapes the scene preview.

- [ ] **Step 3: Commit**
  ```bash
  git add storyboard/index.html
  git commit -m "feat(storyboard): add SVG panel scene generator with character silhouettes"
  ```

---

### Task 11: Panel editor drawer (Card mode form)

**Files:**
- Modify: `storyboard/index.html`

- [ ] **Step 1: Add the editor drawer container in the HTML body**
  Inside `<div class="main-canvas">`, after the panel grid div and before the status bar div, add:
  ```html
  <div class="editor-drawer" id="editor-drawer" style="display:none"></div>
  ```

- [ ] **Step 2: Implement `renderEditor()` and call it from `render()`**
  ```js
  function render() {
    renderSidebar();
    renderTimeline();
    renderViewToggle();
    renderMain();
    renderEditor();
    renderStatusBar();
  }

  function renderEditor() {
    const drawer = document.getElementById('editor-drawer');
    const p = App.project.panels.find(x => x.id === App.ui.selectedPanelId);
    if (!p) { drawer.style.display = 'none'; return; }
    drawer.style.display = 'flex';

    const wordCount = (p.voiceover || '').trim().split(/\s+/).filter(Boolean).length;
    const readTime = Math.ceil(wordCount / 150 * 60);
    const overrun = readTime > p.duration ? 'style="color:var(--red-soft);font-weight:500"' : '';

    const charChecks = App.project.characters.map(c => {
      const on = (p.characters || []).includes(c.id);
      return `<div class="char-ck ${on ? 'on' : ''}" data-char-id="${c.id}" style="${on ? `color:${c.color};border-color:${c.color}66` : ''}">
        <span style="width:6px;height:6px;border-radius:50%;background:${c.color};display:inline-block"></span>${escapeHtml(c.name)}
      </div>`;
    }).join('');

    drawer.innerHTML = `
      <div class="drawer-header">
        <div>
          <div class="drawer-title">Editing Panel ${App.project.panels.indexOf(p) + 1}</div>
          <div class="drawer-subtitle">${escapeHtml(p.label)}</div>
        </div>
        <div style="display:flex;gap:8px">
          <button class="btn btn-ghost" id="drawer-close">Close</button>
          <button class="btn btn-primary" id="drawer-apply">Apply</button>
        </div>
      </div>
      <div class="drawer-body">

        <div class="field">
          <div class="field-label">Label</div>
          <input class="field-input" data-field="label" value="${escapeHtml(p.label)}"/>
          <div class="field-label" style="margin-top:8px">Duration (s)</div>
          <input class="field-input" type="number" data-field="duration" value="${p.duration}"/>
        </div>

        <div class="field">
          <div class="field-label">Shot Type</div>
          <select class="field-select" data-field="shotType">
            ${selectOptions(['wide','medium','close-up','aerial','POV'], p.shotType)}
          </select>
          <div class="field-label" style="margin-top:8px">Camera Angle</div>
          <select class="field-select" data-field="cameraAngle">
            ${selectOptions(['low','eye','high','dutch'], p.cameraAngle)}
          </select>
        </div>

        <div class="field">
          <div class="field-label">Camera Move</div>
          <select class="field-select" data-field="cameraMove">
            ${selectOptions(['static','pan','dolly','handheld','zoom'], p.cameraMove)}
          </select>
          <div class="field-label" style="margin-top:8px">Background</div>
          <select class="field-select" data-field="background">
            ${selectOptions(['alley','office','street','rooftop','studio','room','forest'], p.background)}
          </select>
        </div>

        <div class="field">
          <div class="field-label">Lighting</div>
          <select class="field-select" data-field="lighting">
            ${selectOptions(['dark','bright','neon','sunset','spotlight','silhouette'], p.lighting)}
          </select>
          <div class="field-label" style="margin-top:8px">Emotion</div>
          <select class="field-select" data-field="emotion">
            ${selectOptions(['neutral','tense','excited','sad','mysterious','happy','angry','suspenseful'], p.emotion)}
          </select>
        </div>

        <div class="field">
          <div class="field-label">Characters</div>
          <div class="char-checks">${charChecks || '<span style="color:var(--muted);font-size:0.65rem">Add characters in sidebar</span>'}</div>
          <div class="field-label" style="margin-top:8px">Visual Style</div>
          <select class="field-select" data-field="visualStyle">
            ${selectOptions(['animated-realism','minimal','sketch','comic','cinematic','flat'], p.visualStyle)}
          </select>
        </div>

        <div class="field" style="grid-column: span 2">
          <div class="field-label">Description</div>
          <textarea class="field-textarea" data-field="description">${escapeHtml(p.description)}</textarea>
          <div class="field-label" style="margin-top:6px">Voiceover Script</div>
          <textarea class="field-textarea" data-field="voiceover">${escapeHtml(p.voiceover)}</textarea>
          <div class="field-label" style="margin-top:4px" ${overrun}>${wordCount} words · ~${readTime}s read time${readTime > p.duration ? ' · exceeds panel duration' : ''}</div>
        </div>

      </div>
    `;

    drawer.onclick = null;
    drawer.querySelector('#drawer-close').onclick = () => { App.ui.selectedPanelId = null; render(); };
    drawer.querySelector('#drawer-apply').onclick = () => { saveToStorage(); flashStatus('Saved'); };

    drawer.querySelectorAll('[data-field]').forEach(el => {
      el.oninput = () => {
        const key = el.dataset.field;
        const val = el.type === 'number' ? Number(el.value) : el.value;
        p[key] = val;
        // Light re-render only the cards + status, not the entire drawer (would lose focus)
        renderTimeline(); renderViewToggle(); renderStatusBar();
        // If a card-level field changed, re-render the card grid as well:
        if (['label', 'duration', 'shotType', 'cameraMove', 'emotion', 'lighting', 'description'].includes(key)) {
          if (App.ui.view === 'grid') {
            document.getElementById('panel-grid').innerHTML = renderPanelGrid();
            wirePanelActions(document.getElementById('panel-grid'));
          } else {
            document.getElementById('panel-grid').innerHTML = renderOutlineView();
            wirePanelActions(document.getElementById('panel-grid'));
          }
        }
      };
    });

    drawer.querySelectorAll('[data-char-id]').forEach(el => {
      el.onclick = () => {
        const id = el.dataset.charId;
        p.characters = p.characters || [];
        if (p.characters.includes(id)) p.characters = p.characters.filter(x => x !== id);
        else p.characters.push(id);
        saveToStorage();
        render();
      };
    });
  }

  function selectOptions(opts, selected) {
    return opts.map(o => `<option value="${o}" ${o === selected ? 'selected' : ''}>${capitalize(o)}</option>`).join('');
  }
  ```

- [ ] **Step 2b: Update `wirePanelActions()` so clicking Edit selects panel and the drawer opens**
  Already handled in Task 6's listener — clicking Edit sets `selectedPanelId` and re-renders, which now triggers `renderEditor()`.

- [ ] **Step 3: Verify**
  Add a panel. Click "Edit" on the card → editor drawer slides in at the bottom with all fields populated. Edit the label → card title updates live. Edit the voiceover → word count + read time update under the field; if it exceeds duration, it turns red.

- [ ] **Step 4: Commit**
  ```bash
  git add storyboard/index.html
  git commit -m "feat(storyboard): add panel editor drawer with voiceover read-time"
  ```

---

### Task 12: Templates mode and Sample loader


**Files:**
- Modify: `storyboard/index.html`

- [ ] **Step 1: Define the template library**
  Add this constant near the top of the JS:
  ```js
  const TEMPLATES = [
    { id: 't-wide-establish', label: 'Wide Establishing Shot', shotType: 'wide', cameraMove: 'static', cameraAngle: 'eye', background: 'street', lighting: 'sunset', emotion: 'neutral', duration: 8 },
    { id: 't-talking-head',   label: 'Talking Head',           shotType: 'medium', cameraMove: 'static', cameraAngle: 'eye', background: 'studio', lighting: 'spotlight', emotion: 'neutral', duration: 10 },
    { id: 't-product-reveal', label: 'Product Reveal',         shotType: 'close-up', cameraMove: 'zoom', cameraAngle: 'low', background: 'studio', lighting: 'spotlight', emotion: 'excited', duration: 5 },
    { id: 't-hook-opener',    label: 'Hook Opener',            shotType: 'close-up', cameraMove: 'zoom', cameraAngle: 'eye', background: 'studio', lighting: 'bright', emotion: 'tense', duration: 3 },
    { id: 't-cta-closer',     label: 'CTA Closer',             shotType: 'medium', cameraMove: 'static', cameraAngle: 'eye', background: 'studio', lighting: 'bright', emotion: 'happy', duration: 4 },
    { id: 't-step-callout',   label: 'Tutorial Step',          shotType: 'medium', cameraMove: 'static', cameraAngle: 'high', background: 'studio', lighting: 'bright', emotion: 'neutral', duration: 6 },
  ];
  ```

- [ ] **Step 2: Show the template gallery when activeMode === 'template'**
  Update `renderMain()`:
  ```js
  function renderMain() {
    const main = document.getElementById('panel-grid');
    if (App.ui.activeMode === 'template' && App.project.panels.length > 0) {
      main.innerHTML = renderTemplateGallery() + renderPanelGrid();
      wireTemplateGallery(main);
      wirePanelActions(main);
      return;
    }
    if (App.project.panels.length === 0) { /* ...empty state... */ return; }
    main.innerHTML = App.ui.view === 'outline' ? renderOutlineView() : renderPanelGrid();
    wirePanelActions(main);
  }

  function renderTemplateGallery() {
    return `
      <div class="template-gallery">
        <div class="act-heading">Templates — click to add</div>
        <div class="grid">
          ${TEMPLATES.map(t => `
            <div class="template-card" data-tpl-id="${t.id}">
              <div class="template-name">${escapeHtml(t.label)}</div>
              <div class="template-meta">${t.shotType} · ${t.cameraMove} · ${t.duration}s</div>
            </div>`).join('')}
        </div>
      </div>`;
  }

  function wireTemplateGallery(root) {
    root.querySelectorAll('.template-card').forEach(el => {
      el.onclick = () => {
        const tpl = TEMPLATES.find(t => t.id === el.dataset.tplId);
        if (!tpl) return;
        addPanel();
        const newPanel = App.project.panels[App.project.panels.length - 1];
        Object.assign(newPanel, tpl, { creationMode: 'template', templateId: tpl.id });
        saveToStorage();
        render();
      };
    });
  }
  ```

- [ ] **Step 3: Add CSS for templates**
  ```css
  .template-card { background: var(--surface); border: 1px solid var(--border); border-radius: 10px; padding: 14px; cursor: pointer; transition: all 0.13s; }
  .template-card:hover { border-color: var(--teal); box-shadow: 0 4px 20px rgba(0,0,0,0.05); }
  .template-name { font-family: 'DM Sans', sans-serif; font-size: 0.85rem; font-weight: 600; color: var(--text); margin-bottom: 4px; }
  .template-meta { font-size: 0.65rem; color: var(--muted); text-transform: capitalize; }
  ```

- [ ] **Step 4: Implement `loadSample()`**
  ```js
  function loadSample() {
    if (App.project.panels.length > 0) {
      if (!confirm('Loading the sample will replace your current project. Continue?')) return;
    }
    const heroId = uid();
    const hostId = uid();
    App.project = {
      name: 'Sample Storyboard',
      aspectRatio: '16:9',
      acts: ['Act I — The Hook', 'Act II — The Action'],
      characters: [
        { id: heroId, name: 'Hero',  role: 'hero',  color: '#4A8C8C' },
        { id: hostId, name: 'Host',  role: 'host',  color: '#7B7BA8' },
      ],
      panels: [
        { id: uid(), act: 0, order: 0, creationMode: 'card',     label: 'Wide Establishing — Alley',     shotType: 'wide',     cameraAngle: 'eye',     cameraMove: 'static',   duration: 18, background: 'alley',  lighting: 'neon',      emotion: 'tense',    characters: [heroId, hostId], visualStyle: 'animated-realism', videoStyle: 'cinematic', description: 'Two figures face each other in a rain-soaked alley.', voiceover: 'The night was cold. Two figures stood, separated by silence and rain.', motionNotes: 'Slow dolly push', audioNotes: 'Rain · low drone', sketchDataUrl: null, templateId: null, textPrompt: null },
        { id: uid(), act: 0, order: 1, creationMode: 'template', label: 'ECU Face — Host Close-Up',      shotType: 'close-up', cameraAngle: 'low',     cameraMove: 'static',   duration: 7,  background: 'studio', lighting: 'spotlight', emotion: 'excited',  characters: [hostId],          visualStyle: 'animated-realism', videoStyle: 'youtube',   description: 'Host extreme close-up, low angle.',                voiceover: "Today, we're talking about something that changed everything.", motionNotes: 'Locked',          audioNotes: 'Heartbeat',     sketchDataUrl: null, templateId: 't-talking-head', textPrompt: null },
        { id: uid(), act: 1, order: 2, creationMode: 'text',     label: 'Punch Impact',                  shotType: 'medium',   cameraAngle: 'eye',     cameraMove: 'handheld', duration: 13, background: 'street', lighting: 'dark',      emotion: 'angry',    characters: [heroId, hostId], visualStyle: 'animated-realism', videoStyle: 'cinematic', description: 'Hero right cross connects.',                       voiceover: 'And then it happened in a single moment.',                       motionNotes: 'Camera shake',    audioNotes: 'Bone crack',    sketchDataUrl: null, templateId: null, textPrompt: 'punch impact slow motion' },
        { id: uid(), act: 1, order: 3, creationMode: 'sketch',   label: 'Outro — CTA Sketch',            shotType: 'medium',   cameraAngle: 'eye',     cameraMove: 'zoom',     duration: 9,  background: 'studio', lighting: 'bright',    emotion: 'happy',    characters: [hostId],          visualStyle: 'sketch',           videoStyle: 'youtube',   description: 'Hand-sketched CTA frame.',                         voiceover: 'If this helped you, smash that subscribe button.',                motionNotes: 'Zoom in slight',  audioNotes: 'Upbeat sting',  sketchDataUrl: null, templateId: null, textPrompt: null },
      ],
    };
    App.ui.selectedPanelId = null;
    saveToStorage();
    render();
    flashStatus('Sample loaded');
  }
  ```

- [ ] **Step 5: Verify**
  Click "Load Sample" → 4 panels appear across 2 acts, with 2 characters defined, voiceover scripts, and varied modes/lightings. Switching activeMode to "Templates" while panels exist shows the template gallery above the existing panels; clicking a template adds a new panel.

- [ ] **Step 6: Commit**
  ```bash
  git add storyboard/index.html
  git commit -m "feat(storyboard): add template gallery and sample loader"
  ```

---

### Task 13: Export — JSON, HTML, and PDF


**Files:**
- Modify: `storyboard/index.html`

- [ ] **Step 1: Implement `exportProject(kind)`**
  ```js
  function exportProject(kind) {
    if (App.project.panels.length === 0) {
      alert('Add at least one panel before exporting.');
      return;
    }
    if (kind === 'json')  return exportJson();
    if (kind === 'html')  return exportHtml();
    if (kind === 'pdf')   return exportPdf();
  }

  function downloadBlob(content, filename, mime) {
    const blob = new Blob([content], { type: mime });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url; a.download = filename;
    document.body.appendChild(a); a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  }

  function exportJson() {
    const json = JSON.stringify(App.project, null, 2);
    const safeName = (App.project.name || 'storyboard').replace(/[^a-z0-9]/gi, '-').toLowerCase();
    downloadBlob(json, `${safeName}.json`, 'application/json');
    flashStatus('JSON exported');
  }

  function exportHtml() {
    const safeName = (App.project.name || 'storyboard').replace(/[^a-z0-9]/gi, '-').toLowerCase();
    const styles = document.querySelector('style').innerHTML;
    const panelHtml = App.project.acts.map((name, idx) => `
      <div class="act-heading">${escapeHtml(name)}</div>
      <div class="grid">
        ${App.project.panels.filter(p => p.act === idx).map(renderPanelCard).join('')}
      </div>
    `).join('');
    const html = `<!DOCTYPE html><html><head><meta charset="UTF-8"><title>${escapeHtml(App.project.name)} — Storyboard</title>
      <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600&family=DM+Sans:wght@400;500;600&display=swap" rel="stylesheet">
      <style>${styles}</style></head>
      <body style="background:var(--bg);padding:30px"><h1 style="font-family:'DM Sans',sans-serif;color:var(--text);margin-bottom:24px">${escapeHtml(App.project.name)}</h1>${panelHtml}</body></html>`;
    downloadBlob(html, `${safeName}.html`, 'text/html');
    flashStatus('HTML exported');
  }

  function exportPdf() {
    // Use the browser's print dialog with a temporary print stylesheet
    const original = document.title;
    document.title = `${App.project.name} — Storyboard`;
    document.body.classList.add('printing');
    window.print();
    document.body.classList.remove('printing');
    document.title = original;
  }
  ```

- [ ] **Step 2: Add print stylesheet to the `<style>` block**
  ```css
  @media print {
    .app-header, .sidebar, .timeline-bar, .view-toggle-row, .editor-drawer, .status-bar, .panel-actions, .add-card { display: none !important; }
    body { background: white; }
    .main-canvas { display: block; }
    .panel-grid { padding: 0; }
    .panel-card { break-inside: avoid; page-break-inside: avoid; box-shadow: none; }
    .grid { grid-template-columns: repeat(2, 1fr) !important; gap: 12px; }
  }
  ```

- [ ] **Step 3: Verify**
  Load sample. Click Export → JSON: file downloads. Click Export → HTML: file downloads, opening it in a browser shows the standalone storyboard. Click Export → PDF: print dialog appears with two-column panel layout, no sidebar/header/drawer.

- [ ] **Step 4: Commit**
  ```bash
  git add storyboard/index.html
  git commit -m "feat(storyboard): add JSON, HTML, and PDF export"
  ```

---

### Task 14: Final polish and self-test

**Files:**
- Modify: `storyboard/index.html` (any cleanup)
- Modify: `README.md` (link to new app, optional)

- [ ] **Step 1: Manual smoke test — go through each feature**
  - [ ] Page loads with empty state
  - [ ] Add First Panel works
  - [ ] Edit panel via the drawer; changes persist after reload
  - [ ] Voiceover word count and read time update live; turns red when exceeding duration
  - [ ] Add Character (×2), assign to panel, silhouettes appear in scene
  - [ ] Aspect ratio dropdown reshapes panel scenes
  - [ ] Switch to Outline view; switch back to Grid
  - [ ] Templates mode shows template gallery; clicking adds panel
  - [ ] Load Sample populates 4 panels + 2 characters + 2 acts
  - [ ] Duplicate, delete (with confirm), Ctrl+S all work
  - [ ] Export JSON downloads valid JSON
  - [ ] Export HTML downloads a standalone working file
  - [ ] Export PDF opens print dialog with clean layout
  - [ ] Reload page → previous project restored from LocalStorage

- [ ] **Step 2: Fix any issues found**
  Address bugs surfaced during the smoke test. Keep changes minimal and focused.

- [ ] **Step 3: Add a top-of-file comment with version and date**
  ```html
  <!-- Shot Builder Studio v1.0 (Phase 1) — 2026-05-27 -->
  ```

- [ ] **Step 4: Commit final polish**
  ```bash
  git add storyboard/index.html
  git commit -m "feat(storyboard): Phase 1 polish and version stamp"
  ```

---

## Out of Scope for Phase 1

Per spec §8, deferred to Phase 2 / 3:
- Text-to-Panel keyword SVG generator (basic stub only in Phase 1 — uses the same scene generator as Card mode)
- Sketch mode drawing canvas (placeholder card only — no actual canvas)
- PNG export per panel
- ZIP export of all panels
- Project Manager modal (multi-project support)
- JSON import (drag-drop)

These are intentionally left to keep Phase 1 shippable in one focused effort.

---

## Self-Review Checklist

Before declaring Phase 1 complete:

- [ ] **Spec coverage:** Every Phase 1 item from spec §8 has a matching task above. ✅
- [ ] **No placeholders:** All steps contain actual code or exact commands. ✅
- [ ] **Type consistency:** `App.project.panels[].id` is a string everywhere; `aspectRatio` is `'16:9' | '9:16' | '1:1'` everywhere. ✅
- [ ] **Bite-sized:** Every step is 2-5 minutes of work. ✅
- [ ] **TDD note:** This is a UI-heavy app with no logic that benefits from unit tests. Manual smoke testing in Task 14 covers verification. The user explicitly approved no tests in the goal section. ✅

---

## Execution

This plan is small enough to execute inline. All tasks share state via the single HTML file, so subagent dispatch would add overhead without benefit. Proceed task-by-task in this session.
