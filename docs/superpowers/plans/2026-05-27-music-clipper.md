# Music Clipper Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file browser-based music clipper with waveform visualization, trim/cut/merge, and format export.

**Architecture:** Single `musicclipper/index.html` file. WaveSurfer.js for waveform + region selection. FFmpeg.wasm for audio processing and format conversion. No build step, no server.

**Tech Stack:** WaveSurfer.js 7.x (CDN), FFmpeg.wasm 0.12.x (CDN), Vanilla JS ES2020, Custom CSS

---

### Task 1: HTML Structure & CSS

**Files:**
- Create: `musicclipper/index.html`

- [ ] **Step 1: Create the HTML skeleton**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Music Clipper</title>
</head>
<body>
  <header>
    <h1>🎵 Music Clipper</h1>
    <p>Trim, cut and merge audio — right in your browser</p>
  </header>
  <main>
    <section id="loader-section"><!-- File loader --></section>
    <section id="waveform-section" class="hidden"><!-- Waveform --></section>
    <section id="controls-section" class="hidden"><!-- Time inputs + actions --></section>
    <section id="merge-section"><!-- Merge list --></section>
    <section id="export-section" class="hidden"><!-- Export --></section>
  </main>
  <div id="toast" class="hidden"></div>
</body>
</html>
```

- [ ] **Step 2: Add CSS reset and variables**

```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
:root {
  --bg: #f8f9fa;
  --surface: #ffffff;
  --border: #dee2e6;
  --accent: #4361ee;
  --accent-hover: #3a56d4;
  --danger: #e63946;
  --success: #2a9d8f;
  --text: #212529;
  --text-muted: #6c757d;
  --radius: 10px;
  --shadow: 0 2px 8px rgba(0,0,0,0.08);
}
body { font-family: 'Segoe UI', system-ui, sans-serif; background: var(--bg); color: var(--text); min-height: 100vh; }
.hidden { display: none !important; }
```

- [ ] **Step 3: Style header, sections and cards**

```css
header { text-align: center; padding: 2rem 1rem 1rem; }
header h1 { font-size: 2rem; font-weight: 700; }
header p { color: var(--text-muted); margin-top: 0.25rem; }
main { max-width: 800px; margin: 0 auto; padding: 1rem; display: flex; flex-direction: column; gap: 1.5rem; }
.card { background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 1.5rem; box-shadow: var(--shadow); }
.card h2 { font-size: 1rem; font-weight: 600; margin-bottom: 1rem; color: var(--text-muted); text-transform: uppercase; letter-spacing: 0.05em; }
```

- [ ] **Step 4: Style buttons**

```css
.btn { display: inline-flex; align-items: center; gap: 0.4rem; padding: 0.5rem 1.1rem; border-radius: 6px; border: none; font-size: 0.9rem; font-weight: 500; cursor: pointer; transition: background 0.15s; }
.btn-primary { background: var(--accent); color: #fff; }
.btn-primary:hover { background: var(--accent-hover); }
.btn-danger { background: var(--danger); color: #fff; }
.btn-success { background: var(--success); color: #fff; }
.btn-outline { background: transparent; border: 1px solid var(--border); color: var(--text); }
.btn-outline:hover { background: var(--bg); }
.btn:disabled { opacity: 0.5; cursor: not-allowed; }
```

- [ ] **Step 5: Style toast notification**

```css
#toast { position: fixed; bottom: 1.5rem; left: 50%; transform: translateX(-50%); background: #333; color: #fff; padding: 0.75rem 1.5rem; border-radius: 999px; font-size: 0.9rem; z-index: 9999; animation: fadeIn 0.2s ease; }
@keyframes fadeIn { from { opacity: 0; transform: translateX(-50%) translateY(10px); } to { opacity: 1; transform: translateX(-50%) translateY(0); } }
```

- [ ] **Step 6: Open in browser and verify layout looks clean with no errors**



---

### Task 2: File Loader Section

**Files:**
- Modify: `musicclipper/index.html`

- [ ] **Step 1: Add file loader HTML inside `#loader-section`**

```html
<section id="loader-section" class="card">
  <h2>📂 Load Audio</h2>
  <div id="drop-zone">
    <p>Drag & drop audio files here</p>
    <p class="small">or</p>
    <label class="btn btn-primary" for="file-input">Choose File(s)</label>
    <input type="file" id="file-input" accept="audio/*" multiple hidden/>
  </div>
  <div id="file-list"></div>
</section>
```

- [ ] **Step 2: Add drop zone CSS**

```css
#drop-zone { border: 2px dashed var(--border); border-radius: var(--radius); padding: 2rem; text-align: center; color: var(--text-muted); transition: border-color 0.2s, background 0.2s; }
#drop-zone.dragover { border-color: var(--accent); background: #eef0fd; }
#drop-zone .small { font-size: 0.8rem; margin: 0.4rem 0; }
#file-list { margin-top: 1rem; display: flex; flex-direction: column; gap: 0.5rem; }
.file-item { display: flex; align-items: center; justify-content: space-between; background: var(--bg); border: 1px solid var(--border); border-radius: 6px; padding: 0.5rem 0.75rem; font-size: 0.9rem; }
.file-item button { background: none; border: none; color: var(--danger); cursor: pointer; font-size: 1rem; }
```

- [ ] **Step 3: Add file loader JS**

```js
const fileInput = document.getElementById('file-input');
const dropZone = document.getElementById('drop-zone');
const fileList = document.getElementById('file-list');
let loadedFiles = []; // { name, buffer }

dropZone.addEventListener('dragover', e => { e.preventDefault(); dropZone.classList.add('dragover'); });
dropZone.addEventListener('dragleave', () => dropZone.classList.remove('dragover'));
dropZone.addEventListener('drop', e => { e.preventDefault(); dropZone.classList.remove('dragover'); handleFiles(e.dataTransfer.files); });
fileInput.addEventListener('change', () => handleFiles(fileInput.files));

async function handleFiles(files) {
  for (const file of files) {
    if (!file.type.startsWith('audio/')) { showToast('⚠ Not an audio file: ' + file.name); continue; }
    const buffer = await file.arrayBuffer();
    loadedFiles.push({ name: file.name, buffer });
    renderFileList();
  }
  if (loadedFiles.length === 1) loadFileIntoWaveform(loadedFiles[0]);
}

function renderFileList() {
  fileList.innerHTML = loadedFiles.map((f, i) => `
    <div class="file-item">
      <span>🎵 ${f.name}</span>
      <button onclick="removeFile(${i})">✕</button>
    </div>`).join('');
}

function removeFile(i) {
  loadedFiles.splice(i, 1);
  renderFileList();
  if (loadedFiles.length === 0) hideWaveform();
}
```

- [ ] **Step 4: Verify files can be loaded via drag-drop and file picker, appear in list, and can be removed**



---

### Task 3: Waveform Visualizer & Playback

**Files:**
- Modify: `musicclipper/index.html`

- [ ] **Step 1: Add WaveSurfer.js + Regions plugin via CDN in `<head>`**

```html
<script src="https://unpkg.com/wavesurfer.js@7/dist/wavesurfer.min.js"></script>
<script src="https://unpkg.com/wavesurfer.js@7/dist/plugins/regions.min.js"></script>
```

- [ ] **Step 2: Add waveform HTML inside `#waveform-section`**

```html
<section id="waveform-section" class="card hidden">
  <h2>🎛 Waveform</h2>
  <div id="waveform"></div>
  <div id="playback-controls">
    <button class="btn btn-outline" id="btn-play">▶ Play</button>
    <button class="btn btn-outline" id="btn-stop">⏹ Stop</button>
    <span id="current-time">0:00</span> / <span id="total-time">0:00</span>
  </div>
</section>
```

- [ ] **Step 3: Add waveform CSS**

```css
#waveform { margin: 1rem 0; border-radius: 6px; overflow: hidden; background: var(--bg); }
#playback-controls { display: flex; align-items: center; gap: 0.75rem; margin-top: 0.75rem; }
#current-time, #total-time { font-size: 0.9rem; color: var(--text-muted); font-variant-numeric: tabular-nums; }
```

- [ ] **Step 4: Initialise WaveSurfer and load audio**

```js
let wavesurfer, regionsPlugin, activeRegion;

function loadFileIntoWaveform(file) {
  if (wavesurfer) wavesurfer.destroy();
  regionsPlugin = WaveSurfer.Regions.create();
  wavesurfer = WaveSurfer.create({
    container: '#waveform',
    waveColor: '#4361ee',
    progressColor: '#3a56d4',
    cursorColor: '#e63946',
    height: 100,
    plugins: [regionsPlugin]
  });
  const blob = new Blob([file.buffer], { type: 'audio/*' });
  wavesurfer.loadBlob(blob);
  wavesurfer.on('ready', () => {
    document.getElementById('waveform-section').classList.remove('hidden');
    document.getElementById('controls-section').classList.remove('hidden');
    document.getElementById('export-section').classList.remove('hidden');
    document.getElementById('total-time').textContent = formatTime(wavesurfer.getDuration());
    addDefaultRegion();
  });
  wavesurfer.on('timeupdate', t => {
    document.getElementById('current-time').textContent = formatTime(t);
  });
}

document.getElementById('btn-play').addEventListener('click', () => wavesurfer?.playPause());
document.getElementById('btn-stop').addEventListener('click', () => { wavesurfer?.stop(); });
```

- [ ] **Step 5: Add `formatTime` helper**

```js
function formatTime(s) {
  const m = Math.floor(s / 60);
  const sec = (s % 60).toFixed(2).padStart(5, '0');
  return `${m}:${sec}`;
}
```

- [ ] **Step 6: Verify waveform renders when a file is loaded and playback works**



---

### Task 4: Time Input Controls & Region Sync

**Files:**
- Modify: `musicclipper/index.html`

- [ ] **Step 1: Add controls HTML inside `#controls-section`**

```html
<section id="controls-section" class="card hidden">
  <h2>✂️ Selection</h2>
  <div id="time-inputs">
    <label>Start <input type="text" id="input-start" placeholder="0:00.00"/></label>
    <label>End <input type="text" id="input-end" placeholder="0:00.00"/></label>
    <button class="btn btn-outline" id="btn-select-all">Select All</button>
  </div>
  <div id="action-buttons">
    <button class="btn btn-primary" id="btn-trim">✂️ Trim (keep selection)</button>
    <button class="btn btn-danger" id="btn-cut">🔪 Cut (remove selection)</button>
  </div>
</section>
```

- [ ] **Step 2: Add controls CSS**

```css
#time-inputs { display: flex; align-items: center; gap: 1rem; flex-wrap: wrap; margin-bottom: 1rem; }
#time-inputs label { display: flex; flex-direction: column; font-size: 0.8rem; color: var(--text-muted); gap: 0.25rem; }
#time-inputs input { padding: 0.4rem 0.6rem; border: 1px solid var(--border); border-radius: 6px; font-size: 0.9rem; width: 100px; }
#action-buttons { display: flex; gap: 0.75rem; flex-wrap: wrap; }
```

- [ ] **Step 3: Add default region and bidirectional sync**

```js
function addDefaultRegion() {
  regionsPlugin.clearRegions();
  const dur = wavesurfer.getDuration();
  activeRegion = regionsPlugin.addRegion({
    start: 0, end: dur, color: 'rgba(67,97,238,0.15)', drag: true, resize: true
  });
  syncInputsFromRegion();
  activeRegion.on('update', syncInputsFromRegion);
}

function syncInputsFromRegion() {
  document.getElementById('input-start').value = formatTime(activeRegion.start);
  document.getElementById('input-end').value = formatTime(activeRegion.end);
}

function parseTime(str) {
  const [m, s] = str.split(':');
  return parseFloat(m) * 60 + parseFloat(s || 0);
}

document.getElementById('input-start').addEventListener('change', e => {
  if (!activeRegion) return;
  activeRegion.setOptions({ start: parseTime(e.target.value) });
});
document.getElementById('input-end').addEventListener('change', e => {
  if (!activeRegion) return;
  activeRegion.setOptions({ end: parseTime(e.target.value) });
});
document.getElementById('btn-select-all').addEventListener('click', () => {
  if (!activeRegion) return;
  activeRegion.setOptions({ start: 0, end: wavesurfer.getDuration() });
  syncInputsFromRegion();
});
```

- [ ] **Step 4: Verify dragging the region updates the time inputs and typing in inputs updates the region**



---

### Task 5: FFmpeg.wasm Setup + Trim / Cut / Merge

**Files:**
- Modify: `musicclipper/index.html`

- [ ] **Step 1: Add FFmpeg.wasm CDN scripts in `<head>`**

```html
<script src="https://unpkg.com/@ffmpeg/ffmpeg@0.12.10/dist/umd/ffmpeg.js"></script>
<script src="https://unpkg.com/@ffmpeg/util@0.12.1/dist/umd/index.js"></script>
```

- [ ] **Step 2: Initialise FFmpeg**

```js
const { FFmpeg } = FFmpegWASM;
const { fetchFile } = FFmpegUtil;
const ffmpeg = new FFmpeg();
let ffmpegReady = false;

async function loadFFmpeg() {
  if (ffmpegReady) return;
  showToast('⏳ Loading FFmpeg…');
  await ffmpeg.load();
  ffmpegReady = true;
  showToast('✅ FFmpeg ready');
}
```

- [ ] **Step 3: Add Trim action**

```js
document.getElementById('btn-trim').addEventListener('click', async () => {
  if (!activeRegion) return showToast('⚠ No region selected');
  await loadFFmpeg();
  const { start, end } = activeRegion;
  const file = loadedFiles[0];
  await ffmpeg.writeFile('input', await fetchFile(new Blob([file.buffer])));
  await ffmpeg.exec(['-i','input','-ss',String(start),'-to',String(end),'-c','copy','output_trim']);
  const data = await ffmpeg.readFile('output_trim');
  lastOutput = { data, name: 'trimmed_' + file.name };
  showToast('✅ Trimmed! Choose format and export.');
  document.getElementById('export-section').classList.remove('hidden');
});
```

- [ ] **Step 4: Add Cut action**

```js
document.getElementById('btn-cut').addEventListener('click', async () => {
  if (!activeRegion) return showToast('⚠ No region selected');
  await loadFFmpeg();
  const { start, end } = activeRegion;
  const file = loadedFiles[0];
  const dur = wavesurfer.getDuration();
  await ffmpeg.writeFile('input', await fetchFile(new Blob([file.buffer])));
  // Part before cut
  await ffmpeg.exec(['-i','input','-ss','0','-to',String(start),'-c','copy','part1']);
  // Part after cut
  await ffmpeg.exec(['-i','input','-ss',String(end),'-to',String(dur),'-c','copy','part2']);
  // Concat
  await ffmpeg.writeFile('list.txt', "file 'part1'\nfile 'part2'");
  await ffmpeg.exec(['-f','concat','-safe','0','-i','list.txt','-c','copy','output_cut']);
  const data = await ffmpeg.readFile('output_cut');
  lastOutput = { data, name: 'cut_' + file.name };
  showToast('✅ Cut! Choose format and export.');
});
```

- [ ] **Step 5: Add Merge action**

```js
document.getElementById('btn-merge').addEventListener('click', async () => {
  if (loadedFiles.length < 2) return showToast('⚠ Load at least 2 files to merge');
  await loadFFmpeg();
  let listContent = '';
  for (let i = 0; i < loadedFiles.length; i++) {
    await ffmpeg.writeFile(`merge${i}`, await fetchFile(new Blob([loadedFiles[i].buffer])));
    listContent += `file 'merge${i}'\n`;
  }
  await ffmpeg.writeFile('mergelist.txt', listContent);
  await ffmpeg.exec(['-f','concat','-safe','0','-i','mergelist.txt','-c','copy','output_merge']);
  const data = await ffmpeg.readFile('output_merge');
  lastOutput = { data, name: 'merged.mp3' };
  showToast('✅ Merged! Choose format and export.');
  document.getElementById('export-section').classList.remove('hidden');
});
```

- [ ] **Step 6: Verify trim, cut, and merge each produce output without console errors**



---

### Task 6: Export & Download + Toast Utility

**Files:**
- Modify: `musicclipper/index.html`

- [ ] **Step 1: Add export HTML inside `#export-section`**

```html
<section id="export-section" class="card hidden">
  <h2>💾 Export</h2>
  <div id="export-controls">
    <label>Format
      <select id="format-select">
        <option value="mp3">MP3</option>
        <option value="wav">WAV</option>
        <option value="ogg">OGG</option>
      </select>
    </label>
    <button class="btn btn-success" id="btn-export">⬇️ Export & Download</button>
  </div>
  <div id="progress-bar" class="hidden">
    <div id="progress-fill"></div>
    <span id="progress-label">Processing…</span>
  </div>
</section>
```

- [ ] **Step 2: Add export CSS**

```css
#export-controls { display: flex; align-items: center; gap: 1rem; flex-wrap: wrap; }
#export-controls select { padding: 0.4rem 0.6rem; border: 1px solid var(--border); border-radius: 6px; font-size: 0.9rem; }
#progress-bar { margin-top: 1rem; background: var(--bg); border-radius: 999px; overflow: hidden; height: 8px; position: relative; }
#progress-fill { height: 100%; background: var(--accent); width: 0%; transition: width 0.3s; }
#progress-label { font-size: 0.8rem; color: var(--text-muted); display: block; margin-top: 0.4rem; }
```

- [ ] **Step 3: Add export & download JS**

```js
let lastOutput = null;

document.getElementById('btn-export').addEventListener('click', async () => {
  if (!lastOutput) return showToast('⚠ Nothing to export yet — trim, cut or merge first');
  await loadFFmpeg();
  const format = document.getElementById('format-select').value;
  const mimeMap = { mp3: 'audio/mpeg', wav: 'audio/wav', ogg: 'audio/ogg' };
  document.getElementById('progress-bar').classList.remove('hidden');
  ffmpeg.on('progress', ({ progress }) => {
    document.getElementById('progress-fill').style.width = (progress * 100) + '%';
    document.getElementById('progress-label').textContent = `Processing… ${Math.round(progress * 100)}%`;
  });
  await ffmpeg.writeFile('export_input', lastOutput.data);
  await ffmpeg.exec(['-i', 'export_input', `out.${format}`]);
  const outData = await ffmpeg.readFile(`out.${format}`);
  const blob = new Blob([outData.buffer], { type: mimeMap[format] });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = lastOutput.name.replace(/\.[^.]+$/, '') + '.' + format;
  a.click();
  URL.revokeObjectURL(url);
  document.getElementById('progress-bar').classList.add('hidden');
  showToast('✅ Downloaded!');
});
```

- [ ] **Step 4: Add toast utility**

```js
function showToast(msg, duration = 3000) {
  const t = document.getElementById('toast');
  t.textContent = msg;
  t.classList.remove('hidden');
  clearTimeout(t._timer);
  t._timer = setTimeout(() => t.classList.add('hidden'), duration);
}
```

- [ ] **Step 5: Add merge section HTML (for the merge button)**

```html
<section id="merge-section" class="card">
  <h2>🔗 Merge Clips</h2>
  <p style="color:var(--text-muted);font-size:0.9rem;margin-bottom:1rem">Load 2+ files above, then merge them in order.</p>
  <button class="btn btn-primary" id="btn-merge">🔗 Merge All Files</button>
</section>
```

- [ ] **Step 6: Add `hideWaveform` helper**

```js
function hideWaveform() {
  document.getElementById('waveform-section').classList.add('hidden');
  document.getElementById('controls-section').classList.add('hidden');
  document.getElementById('export-section').classList.add('hidden');
}
```

- [ ] **Step 7: Verify full flow — load file → trim/cut → choose format → download works end to end**

- [ ] **Step 8: Commit**

```bash
git add musicclipper/index.html docs/superpowers/specs/2026-05-27-music-clipper-design.md docs/superpowers/plans/2026-05-27-music-clipper.md
git commit -m "feat: add Music Clipper app with trim, cut, merge and format export"
```
