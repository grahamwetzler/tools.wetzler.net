# dbt Run Results Visualizer Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a single-file browser tool at `/dbt-run-results/` that visualizes dbt `run_results.json` as a thread-by-thread Gantt chart with bottleneck highlighting.

**Architecture:** Single HTML file with inline CSS and JS. File uploaded client-side via FileReader, parsed with native JSON.parse, rendered onto HTML5 Canvas. Tooltip uses a floating `<div>` positioned on mousemove.

**Tech Stack:** Vanilla HTML/CSS/JS, HTML5 Canvas API, DM Mono font via Google Fonts CDN (same as homepage).

---

### Task 1: Scaffold the page shell

**Files:**
- Create: `dbt-run-results/index.html`
- Modify: `index.html` (homepage — add tool card)

**Step 1: Create the directory and empty file**

```bash
mkdir -p dbt-run-results
touch dbt-run-results/index.html
```

**Step 2: Write the HTML shell**

Create `dbt-run-results/index.html` with this content:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>dbt run results — tools.wetzler.net</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=DM+Mono:wght@300;400;500&display=swap" rel="stylesheet">
  <style>
    /* styles go here in Task 2 */
  </style>
</head>
<body>
  <header>
    <h1>// <a href="/">tools.wetzler.net</a> / <span>dbt-run-results</span></h1>
    <p class="subtitle">visualize dbt run_results.json as a thread timeline</p>
  </header>

  <main>
    <!-- upload zone — Task 3 -->
    <!-- chart area — Task 4 -->
    <!-- legend — Task 6 -->
    <!-- tooltip div — Task 5 -->
  </main>

  <script>
    // JS goes here — Tasks 3–7
  </script>
</body>
</html>
```

**Step 3: Add tool card to homepage**

In `index.html`, find the `<ul class="tool-list">` and add a new `<li>` after the `/geo` entry:

```html
      <li>
        <a href="/dbt-run-results/">
          <span class="tool-path">/dbt-run-results</span>
          <span class="tool-desc">visualize dbt run_results.json as a thread timeline gantt chart</span>
        </a>
      </li>
```

**Step 4: Verify by opening in browser**

Open `dbt-run-results/index.html` in a browser. Expect: blank page with title showing. No errors in console.

**Step 5: Commit**

```bash
git add dbt-run-results/index.html index.html
git commit -m "feat: scaffold dbt-run-results tool and add homepage card"
```

---

### Task 2: Add CSS styles

**Files:**
- Modify: `dbt-run-results/index.html` (fill in the `<style>` block)

**Step 1: Write the styles**

Replace the `/* styles go here */` comment in the `<style>` block with:

```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

:root {
  --bg: #0f0f0f;
  --surface: #1a1a1a;
  --border: #2a2a2a;
  --text: #e0e0e0;
  --muted: #666;
  --accent: #7c9fff;
}

body {
  background: var(--bg);
  color: var(--text);
  font-family: 'DM Mono', monospace;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  padding: 48px 24px;
  gap: 32px;
}

header {
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
}

header h1 {
  font-size: 0.9rem;
  font-weight: 400;
  color: var(--muted);
  letter-spacing: 0.05em;
}

header h1 a { color: var(--muted); text-decoration: none; }
header h1 a:hover { color: var(--accent); }
header h1 span { color: var(--text); }

.subtitle {
  margin-top: 8px;
  font-size: 0.8rem;
  font-weight: 300;
  color: var(--muted);
}

main {
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 24px;
}

/* Upload zone */
.upload-zone {
  border: 1px dashed var(--border);
  border-radius: 4px;
  padding: 48px 24px;
  text-align: center;
  cursor: pointer;
  transition: border-color 0.15s, background 0.15s;
  color: var(--muted);
  font-size: 0.85rem;
}

.upload-zone:hover,
.upload-zone.drag-over {
  border-color: var(--accent);
  background: rgba(124, 159, 255, 0.04);
  color: var(--text);
}

.upload-zone input[type="file"] { display: none; }

/* Error message */
.error-msg {
  color: #ff6b6b;
  font-size: 0.8rem;
  display: none;
}

.error-msg.visible { display: block; }

/* Chart container */
.chart-container {
  display: none;
  flex-direction: column;
  gap: 0;
}

.chart-container.visible { display: flex; }

.chart-header {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  margin-bottom: 12px;
}

.chart-title {
  font-size: 0.85rem;
  color: var(--text);
}

.chart-meta {
  font-size: 0.75rem;
  color: var(--muted);
}

/* Canvas wrapper — threads scroll, axis sticks */
.canvas-wrapper {
  position: relative;
  border: 1px solid var(--border);
  border-radius: 4px;
  overflow: hidden;
}

.canvas-scroll {
  overflow-y: auto;
  overflow-x: hidden;
  max-height: 600px;
}

canvas#chart {
  display: block;
  width: 100%;
}

canvas#axis {
  display: block;
  width: 100%;
  border-top: 1px solid var(--border);
}

/* Tooltip */
#tooltip {
  position: fixed;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 4px;
  padding: 10px 12px;
  font-size: 0.75rem;
  color: var(--text);
  pointer-events: none;
  display: none;
  z-index: 100;
  max-width: 280px;
  line-height: 1.6;
}

#tooltip.visible { display: block; }

.tt-name { font-weight: 500; margin-bottom: 4px; word-break: break-all; }
.tt-badges { display: flex; gap: 6px; margin-bottom: 6px; flex-wrap: wrap; }
.tt-badge {
  font-size: 0.7rem;
  padding: 1px 6px;
  border-radius: 2px;
  background: var(--bg);
  border: 1px solid var(--border);
}
.tt-row { color: var(--muted); }
.tt-row span { color: var(--text); }
.tt-bottleneck { color: #ffd700; margin-top: 4px; }

/* Legend */
.legend {
  display: none;
  flex-wrap: wrap;
  gap: 16px;
  font-size: 0.75rem;
  color: var(--muted);
  padding-top: 8px;
}

.legend.visible { display: flex; }

.legend-item {
  display: flex;
  align-items: center;
  gap: 6px;
}

.legend-swatch {
  width: 14px;
  height: 14px;
  border-radius: 2px;
  flex-shrink: 0;
}
```

**Step 2: Verify in browser**

Reload the page. Expect: dark background, header visible, styled upload zone in the center area. No console errors.

**Step 3: Commit**

```bash
git add dbt-run-results/index.html
git commit -m "feat: add css styles for dbt-run-results tool"
```

---

### Task 3: HTML structure + file upload logic

**Files:**
- Modify: `dbt-run-results/index.html`

**Step 1: Add HTML structure to `<main>`**

Replace the comment placeholders in `<main>` with:

```html
  <main>
    <div class="upload-zone" id="uploadZone">
      <input type="file" id="fileInput" accept=".json,application/json">
      <p>drag & drop <code>run_results.json</code> here, or <span style="color:var(--accent);cursor:pointer">click to browse</span></p>
    </div>

    <p class="error-msg" id="errorMsg"></p>

    <div class="chart-container" id="chartContainer">
      <div class="chart-header">
        <span class="chart-title" id="chartTitle">run results</span>
        <span class="chart-meta" id="chartMeta"></span>
      </div>
      <div class="canvas-wrapper">
        <div class="canvas-scroll" id="canvasScroll">
          <canvas id="chart"></canvas>
        </div>
        <canvas id="axis" height="36"></canvas>
      </div>
    </div>

    <div class="legend" id="legend"></div>

    <div id="tooltip">
      <div class="tt-name" id="ttName"></div>
      <div class="tt-badges" id="ttBadges"></div>
      <div class="tt-row">duration: <span id="ttDuration"></span></div>
      <div class="tt-row" id="ttCompileRow">compile: <span id="ttCompile"></span></div>
      <div class="tt-row" id="ttExecuteRow">execute: <span id="ttExecute"></span></div>
      <div class="tt-bottleneck" id="ttBottleneck"></div>
    </div>
  </main>
```

**Step 2: Add file upload JS**

Inside the `<script>` tag, add:

```js
// --- State ---
let barRects = []; // [{node, x, y, w, h}] for hit testing

// --- DOM refs ---
const uploadZone = document.getElementById('uploadZone');
const fileInput  = document.getElementById('fileInput');
const errorMsg   = document.getElementById('errorMsg');

// --- Upload zone interactions ---
uploadZone.addEventListener('click', () => fileInput.click());

uploadZone.addEventListener('dragover', e => {
  e.preventDefault();
  uploadZone.classList.add('drag-over');
});

uploadZone.addEventListener('dragleave', () => {
  uploadZone.classList.remove('drag-over');
});

uploadZone.addEventListener('drop', e => {
  e.preventDefault();
  uploadZone.classList.remove('drag-over');
  const file = e.dataTransfer.files[0];
  if (file) loadFile(file);
});

fileInput.addEventListener('change', e => {
  if (e.target.files[0]) loadFile(e.target.files[0]);
});

function showError(msg) {
  errorMsg.textContent = msg;
  errorMsg.classList.add('visible');
}

function clearError() {
  errorMsg.textContent = '';
  errorMsg.classList.remove('visible');
}

function loadFile(file) {
  clearError();
  const reader = new FileReader();
  reader.onload = e => {
    try {
      const data = JSON.parse(e.target.result);
      processRunResults(data);
    } catch (err) {
      showError('Failed to parse JSON: ' + err.message);
    }
  };
  reader.onerror = () => showError('Failed to read file.');
  reader.readAsText(file);
}
```

**Step 3: Verify in browser**

Reload. Click the upload zone — file picker should open. Drag a file over — border should turn blue. No console errors.

**Step 4: Commit**

```bash
git add dbt-run-results/index.html
git commit -m "feat: add upload zone html and file reader logic"
```

---

### Task 4: Data parsing — processRunResults()

**Files:**
- Modify: `dbt-run-results/index.html` (add to `<script>`)

**Step 1: Add the constants and parser**

Append to the `<script>` block:

```js
// --- Constants ---
const NODE_COLORS = {
  model:    '#7c9fff',
  test:     '#6fcf97',
  snapshot: '#bb8fce',
  seed:     '#e6a96a',
  source:   '#5bb8c4',
  analysis: '#f0d080',
  default:  '#888888',
};

const ROW_H  = 32;  // px per thread row
const ROW_GAP = 4;  // px gap between rows
const LABEL_W = 88; // px for thread label column

function parseTimestamp(s) {
  // ISO 8601 string -> ms since epoch
  return new Date(s).getTime();
}

function getNodeType(uniqueId) {
  // unique_id format: "model.project.name" or "test.project.name"
  const prefix = uniqueId.split('.')[0];
  return prefix || 'default';
}

function getNodeName(uniqueId) {
  // Strip "type.project." prefix, return just the node name
  const parts = uniqueId.split('.');
  return parts.length >= 3 ? parts.slice(2).join('.') : uniqueId;
}

function getTimingBreakdown(timingArr) {
  let compile = null, execute = null;
  for (const t of (timingArr || [])) {
    const dur = (parseTimestamp(t.completed_at) - parseTimestamp(t.started_at)) / 1000;
    if (t.name === 'compile') compile = dur;
    if (t.name === 'execute') execute = dur;
  }
  return { compile, execute };
}

function processRunResults(data) {
  // Validate structure
  if (!data.results || !Array.isArray(data.results)) {
    showError('Invalid run_results.json: missing "results" array.');
    return;
  }

  // Parse each result into a node object
  const nodes = [];
  for (const r of data.results) {
    if (!r.timing || r.timing.length === 0) continue; // skip untimed nodes

    const startedAt  = Math.min(...r.timing.map(t => parseTimestamp(t.started_at)));
    const completedAt = Math.max(...r.timing.map(t => parseTimestamp(t.completed_at)));

    if (!isFinite(startedAt) || !isFinite(completedAt)) continue;

    const { compile, execute } = getTimingBreakdown(r.timing);

    nodes.push({
      uniqueId:      r.unique_id,
      name:          getNodeName(r.unique_id),
      type:          getNodeType(r.unique_id),
      status:        r.status || 'success',
      threadId:      r.thread_id || 'thread-1',
      startedAt,
      completedAt,
      executionTime: r.execution_time || (completedAt - startedAt) / 1000,
      compile,
      execute,
    });
  }

  if (nodes.length === 0) {
    showError('No timed results found in this file.');
    return;
  }

  // Compute run-level stats
  const runStart    = Math.min(...nodes.map(n => n.startedAt));
  const runEnd      = Math.max(...nodes.map(n => n.completedAt));
  const totalDurMs  = runEnd - runStart;

  // Sort threads: natural sort on thread id string
  const threadIds = [...new Set(nodes.map(n => n.threadId))].sort((a, b) => {
    return a.localeCompare(b, undefined, { numeric: true });
  });

  // Determine bottleneck threshold (top 10% by execution_time)
  const times = nodes.map(n => n.executionTime).sort((a, b) => a - b);
  const p90   = times[Math.floor(times.length * 0.9)];

  // Determine critical thread (thread whose last node finishes latest)
  const threadEnds = {};
  for (const n of nodes) {
    if (!threadEnds[n.threadId] || n.completedAt > threadEnds[n.threadId]) {
      threadEnds[n.threadId] = n.completedAt;
    }
  }
  const criticalThread = Object.entries(threadEnds).sort((a, b) => b[1] - a[1])[0][0];

  // Attach computed flags to nodes
  for (const n of nodes) {
    n.isBottleneck    = n.executionTime >= p90;
    n.isCritical      = n.threadId === criticalThread;
  }

  // Update chart header
  const totalSec = (totalDurMs / 1000).toFixed(1);
  const nodeCount = nodes.length;
  document.getElementById('chartTitle').textContent =
    data.metadata?.generated_at
      ? 'run on ' + new Date(data.metadata.generated_at).toLocaleString()
      : 'run results';
  document.getElementById('chartMeta').textContent =
    `${nodeCount} nodes · ${threadIds.length} threads · ${totalSec}s total`;

  // Show chart area
  document.getElementById('chartContainer').classList.add('visible');
  document.getElementById('legend').classList.add('visible');

  renderChart({ nodes, threadIds, runStart, totalDurMs, criticalThread });
  renderLegend();
  setupTooltip(nodes);
}
```

**Step 2: Test by loading a sample file**

Create a minimal test file `test-sample.json`:

```json
{
  "metadata": { "generated_at": "2024-01-15T14:32:00Z" },
  "results": [
    {
      "unique_id": "model.my_project.stg_orders",
      "status": "success",
      "thread_id": "Thread-1",
      "execution_time": 12.4,
      "timing": [
        { "name": "compile", "started_at": "2024-01-15T14:30:00Z", "completed_at": "2024-01-15T14:30:00.3Z" },
        { "name": "execute", "started_at": "2024-01-15T14:30:00.3Z", "completed_at": "2024-01-15T14:30:12.4Z" }
      ]
    },
    {
      "unique_id": "test.my_project.unique_stg_orders_id",
      "status": "pass",
      "thread_id": "Thread-2",
      "execution_time": 2.1,
      "timing": [
        { "name": "compile", "started_at": "2024-01-15T14:30:01Z", "completed_at": "2024-01-15T14:30:01.2Z" },
        { "name": "execute", "started_at": "2024-01-15T14:30:01.2Z", "completed_at": "2024-01-15T14:30:03.1Z" }
      ]
    }
  ]
}
```

Upload this file. Expected: no error, chart area becomes visible, meta shows "2 nodes · 2 threads".

**Step 3: Commit**

```bash
git add dbt-run-results/index.html
git commit -m "feat: add run_results.json parser and data model"
```

---

### Task 5: Render the Gantt chart on Canvas

**Files:**
- Modify: `dbt-run-results/index.html` (add to `<script>`)

**Step 1: Add renderChart() function**

Append to the `<script>` block:

```js
function renderChart({ nodes, threadIds, runStart, totalDurMs, criticalThread }) {
  const canvas = document.getElementById('chart');
  const wrapper = document.getElementById('canvasScroll');

  // Canvas dimensions — use wrapper width, compute height from thread count
  const W = wrapper.clientWidth || 800;
  const totalH = threadIds.length * (ROW_H + ROW_GAP);

  canvas.width  = W;
  canvas.height = totalH;
  canvas.style.width  = W + 'px';
  canvas.style.height = totalH + 'px';

  const ctx = canvas.getContext('2d');
  ctx.clearRect(0, 0, W, totalH);

  // Build a hatch pattern for skipped nodes
  const hatchCanvas = document.createElement('canvas');
  hatchCanvas.width = hatchCanvas.height = 8;
  const hctx = hatchCanvas.getContext('2d');
  hctx.strokeStyle = 'rgba(255,255,255,0.15)';
  hctx.lineWidth = 1;
  hctx.beginPath(); hctx.moveTo(0, 8); hctx.lineTo(8, 0); hctx.stroke();
  const hatch = ctx.createPattern(hatchCanvas, 'repeat');

  barRects = []; // reset hit-test rects

  // Thread label font
  ctx.font = '500 11px DM Mono, monospace';

  for (let i = 0; i < threadIds.length; i++) {
    const tid = threadIds[i];
    const rowY = i * (ROW_H + ROW_GAP);
    const isCriticalRow = tid === criticalThread;

    // Draw thread label
    ctx.fillStyle = isCriticalRow ? '#7c9fff' : '#666';
    ctx.fillText(
      (isCriticalRow ? '★ ' : '') + tid.replace('Thread-', 't-'),
      4,
      rowY + ROW_H / 2 + 4
    );

    // Draw bars for this thread
    const threadNodes = nodes.filter(n => n.threadId === tid);

    for (const node of threadNodes) {
      const baseColor = NODE_COLORS[node.type] || NODE_COLORS.default;
      const offsetMs  = node.startedAt - runStart;
      const barX      = LABEL_W + (offsetMs / totalDurMs) * (W - LABEL_W);
      const barW      = Math.max(2, (node.executionTime * 1000 / totalDurMs) * (W - LABEL_W));
      const barY      = rowY + 2;
      const barH      = ROW_H - 4;

      // Fill
      const isSkip = node.status === 'skip';
      const isError = node.status === 'error' || node.status === 'fail';
      const isWarn  = node.status === 'warn';

      if (isSkip) {
        ctx.globalAlpha = 0.3;
        ctx.fillStyle = baseColor;
        ctx.fillRect(barX, barY, barW, barH);
        ctx.globalAlpha = 1;
        ctx.fillStyle = hatch;
        ctx.fillRect(barX, barY, barW, barH);
      } else if (isError) {
        // Base color with red tint
        ctx.fillStyle = baseColor;
        ctx.globalAlpha = node.isBottleneck ? 1 : 0.85;
        ctx.fillRect(barX, barY, barW, barH);
        ctx.globalAlpha = 0.35;
        ctx.fillStyle = '#ff4444';
        ctx.fillRect(barX, barY, barW, barH);
        ctx.globalAlpha = 1;
      } else {
        ctx.globalAlpha = node.isBottleneck ? 1 : 0.82;
        ctx.fillStyle = baseColor;
        ctx.fillRect(barX, barY, barW, barH);
        ctx.globalAlpha = 1;
      }

      // Status border (top edge, 2px)
      if (isError) {
        ctx.fillStyle = '#ff4444';
        ctx.fillRect(barX, barY, barW, 2);
      } else if (isWarn) {
        ctx.fillStyle = '#ffd700';
        ctx.fillRect(barX, barY, barW, 2);
      }

      // Bottleneck glyph
      if (node.isBottleneck && barW > 16) {
        ctx.font = '11px sans-serif';
        ctx.fillStyle = '#fff';
        ctx.globalAlpha = 0.9;
        ctx.fillText('⚡', barX + 3, barY + barH / 2 + 4);
        ctx.globalAlpha = 1;
        ctx.font = '500 11px DM Mono, monospace';
      }

      // Store rect for hit-testing
      barRects.push({ node, x: barX, y: barY, w: barW, h: barH });
    }
  }

  renderAxis({ W, totalDurMs });
}

function renderAxis({ W, totalDurMs }) {
  const axisCanvas = document.getElementById('axis');
  axisCanvas.width  = W;
  axisCanvas.style.width = W + 'px';

  const ctx = axisCanvas.getContext('2d');
  ctx.clearRect(0, 0, W, 36);

  ctx.fillStyle = '#0f0f0f';
  ctx.fillRect(0, 0, W, 36);

  // Draw tick marks and labels
  const chartW = W - LABEL_W;
  const totalSec = totalDurMs / 1000;

  // Choose a nice tick interval
  const targetTicks = Math.max(4, Math.floor(chartW / 80));
  const rawInterval = totalSec / targetTicks;
  const niceIntervals = [0.5, 1, 2, 5, 10, 15, 30, 60, 120, 300, 600];
  const tickInterval = niceIntervals.find(n => n >= rawInterval) || rawInterval;

  ctx.fillStyle = '#666';
  ctx.font = '10px DM Mono, monospace';
  ctx.textAlign = 'center';

  for (let t = 0; t <= totalSec; t += tickInterval) {
    const x = LABEL_W + (t / totalSec) * chartW;
    ctx.fillStyle = '#2a2a2a';
    ctx.fillRect(x, 0, 1, 8);
    ctx.fillStyle = '#666';
    ctx.fillText(formatDur(t), x, 24);
    if (t + tickInterval > totalSec && t < totalSec) {
      // Also draw end tick
      const endX = LABEL_W + chartW;
      ctx.fillStyle = '#2a2a2a';
      ctx.fillRect(endX, 0, 1, 8);
      ctx.fillStyle = '#666';
      ctx.fillText(formatDur(totalSec), endX, 24);
    }
  }
}

function formatDur(sec) {
  if (sec < 60) return sec.toFixed(sec < 10 ? 1 : 0) + 's';
  return Math.floor(sec / 60) + 'm' + (sec % 60 ? Math.floor(sec % 60) + 's' : '');
}

// Re-render on resize
window.addEventListener('resize', () => {
  if (barRects.length === 0) return;
  // Re-trigger from stored data — not ideal but sufficient for rare resize
  // Full re-render handled by storing last render args
});
```

**Step 2: Test with sample file**

Upload `test-sample.json`. Expected: a canvas appears with two colored bars — one blue (model) on thread-1 row, one green (test) on thread-2 row. A time axis below shows `0s` and elapsed time labels.

**Step 3: Commit**

```bash
git add dbt-run-results/index.html
git commit -m "feat: render gantt chart bars and time axis on canvas"
```

---

### Task 6: Tooltip on hover

**Files:**
- Modify: `dbt-run-results/index.html` (add to `<script>`)

**Step 1: Add setupTooltip() and mousemove handler**

Append to the `<script>` block:

```js
function setupTooltip(nodes) {
  const canvas  = document.getElementById('chart');
  const tooltip = document.getElementById('tooltip');

  canvas.addEventListener('mousemove', e => {
    const rect  = canvas.getBoundingClientRect();
    const mx    = e.clientX - rect.left;
    const my    = e.clientY - rect.top + document.getElementById('canvasScroll').scrollTop;

    const hit = barRects.find(b => mx >= b.x && mx <= b.x + b.w && my >= b.y && my <= b.y + b.h);

    if (hit) {
      const n = hit.node;
      document.getElementById('ttName').textContent = n.name;

      // Badges
      const badgesEl = document.getElementById('ttBadges');
      badgesEl.innerHTML = '';
      const typeBadge = document.createElement('span');
      typeBadge.className = 'tt-badge';
      typeBadge.textContent = n.type;
      typeBadge.style.borderColor = NODE_COLORS[n.type] || '#888';
      typeBadge.style.color = NODE_COLORS[n.type] || '#888';
      badgesEl.appendChild(typeBadge);

      const statusBadge = document.createElement('span');
      statusBadge.className = 'tt-badge';
      statusBadge.textContent = n.status;
      badgesEl.appendChild(statusBadge);

      document.getElementById('ttDuration').textContent = n.executionTime.toFixed(2) + 's';

      const compileRow  = document.getElementById('ttCompileRow');
      const executeRow  = document.getElementById('ttExecuteRow');
      if (n.compile !== null) {
        compileRow.style.display = '';
        document.getElementById('ttCompile').textContent = n.compile.toFixed(2) + 's';
      } else {
        compileRow.style.display = 'none';
      }
      if (n.execute !== null) {
        executeRow.style.display = '';
        document.getElementById('ttExecute').textContent = n.execute.toFixed(2) + 's';
      } else {
        executeRow.style.display = 'none';
      }

      const btEl = document.getElementById('ttBottleneck');
      btEl.textContent = n.isBottleneck ? '⚡ bottleneck (top 10% by duration)' : '';

      // Position tooltip near cursor, keep in viewport
      const tx = e.clientX + 14;
      const ty = e.clientY + 14;
      tooltip.style.left = tx + 'px';
      tooltip.style.top  = ty + 'px';
      tooltip.classList.add('visible');
    } else {
      tooltip.classList.remove('visible');
    }
  });

  canvas.addEventListener('mouseleave', () => {
    document.getElementById('tooltip').classList.remove('visible');
  });
}
```

**Step 2: Test tooltip**

Upload sample file. Hover over a bar. Expected: tooltip appears near cursor showing node name, type badge (colored), status badge, duration, compile/execute split. Moving off the bar hides tooltip.

**Step 3: Commit**

```bash
git add dbt-run-results/index.html
git commit -m "feat: add hover tooltip with node details and bottleneck indicator"
```

---

### Task 7: Render legend

**Files:**
- Modify: `dbt-run-results/index.html` (add to `<script>`)

**Step 1: Add renderLegend() function**

Append to the `<script>` block:

```js
function renderLegend() {
  const legend = document.getElementById('legend');
  legend.innerHTML = '';

  const types = Object.entries(NODE_COLORS).filter(([k]) => k !== 'default');
  for (const [type, color] of types) {
    const item = document.createElement('div');
    item.className = 'legend-item';
    item.innerHTML = `
      <div class="legend-swatch" style="background:${color}"></div>
      <span>${type}</span>
    `;
    legend.appendChild(item);
  }

  // Status modifiers
  const sep = document.createElement('span');
  sep.style.cssText = 'color:var(--border);margin:0 4px';
  sep.textContent = '|';
  legend.appendChild(sep);

  const statuses = [
    { label: 'error/fail', style: 'background:#7c9fff;border-top:2px solid #ff4444' },
    { label: 'warn',       style: 'background:#7c9fff;border-top:2px solid #ffd700' },
    { label: 'skip',       style: 'background:#7c9fff;opacity:0.3' },
    { label: '⚡ bottleneck', style: 'background:#7c9fff;filter:brightness(1.2)' },
    { label: '★ critical thread', style: 'background:none;color:var(--accent)' },
  ];

  for (const s of statuses) {
    const item = document.createElement('div');
    item.className = 'legend-item';
    if (s.label.startsWith('★')) {
      item.innerHTML = `<span style="color:var(--accent)">★</span><span>${s.label.slice(2)}</span>`;
    } else {
      item.innerHTML = `
        <div class="legend-swatch" style="${s.style}"></div>
        <span>${s.label}</span>
      `;
    }
    legend.appendChild(item);
  }
}
```

**Step 2: Test legend**

Upload sample file. Expected: legend appears below chart with color swatches for each node type and status modifier examples.

**Step 3: Commit**

```bash
git add dbt-run-results/index.html
git commit -m "feat: add legend for node types and status modifiers"
```

---

### Task 8: Handle resize and store render state

**Files:**
- Modify: `dbt-run-results/index.html` (update the resize handler placeholder)

**Step 1: Store last render args and re-render on resize**

Find the `window.addEventListener('resize', ...)` block added in Task 5 and replace it with:

```js
let lastRenderArgs = null;

window.addEventListener('resize', () => {
  if (!lastRenderArgs) return;
  renderChart(lastRenderArgs);
});
```

Then in `renderChart()`, add as the first line inside the function body:

```js
lastRenderArgs = { nodes, threadIds, runStart, totalDurMs, criticalThread };
```

**Step 2: Test resize**

Upload sample file, then resize the browser window. Expected: chart redraws correctly at new width.

**Step 3: Commit**

```bash
git add dbt-run-results/index.html
git commit -m "feat: re-render chart on window resize"
```

---

### Task 9: End-to-end test with a real or large sample file

**Step 1: Generate a larger test fixture**

Paste this into the browser console to generate a 200-node test file and download it:

```js
const results = [];
const types = ['model', 'test', 'snapshot', 'seed'];
const statuses = ['success', 'success', 'success', 'error', 'warn', 'skip'];
const base = new Date('2024-01-15T14:00:00Z').getTime();
let cursor = base;

for (let i = 0; i < 200; i++) {
  const thread = `Thread-${(i % 6) + 1}`;
  const type = types[i % types.length];
  const status = statuses[i % statuses.length];
  const dur = (Math.random() * 30 + 0.1) * 1000; // 0.1–30s in ms
  const start = new Date(cursor + Math.random() * 5000).toISOString();
  const end = new Date(cursor + Math.random() * 5000 + dur).toISOString();
  const compileDur = dur * 0.03;
  results.push({
    unique_id: `${type}.my_project.node_${i}`,
    status,
    thread_id: thread,
    execution_time: dur / 1000,
    timing: [
      { name: 'compile', started_at: start, completed_at: new Date(new Date(start).getTime() + compileDur).toISOString() },
      { name: 'execute', started_at: new Date(new Date(start).getTime() + compileDur).toISOString(), completed_at: end },
    ]
  });
  cursor += dur * 0.2;
}

const blob = new Blob([JSON.stringify({ metadata: { generated_at: new Date().toISOString() }, results })], {type: 'application/json'});
const a = document.createElement('a'); a.href = URL.createObjectURL(blob); a.download = 'test-large.json'; a.click();
```

**Step 2: Upload the generated file**

Upload `test-large.json`. Expected:
- Chart renders 200 nodes across 6 threads
- Some bars are red-tinted (errors), yellow-bordered (warn), hatched (skip)
- Top ~20 nodes by duration have ⚡ glyph
- Critical thread row label is blue with ★
- Tooltip works on hover
- Resize redraws correctly
- No console errors

**Step 3: Commit if everything looks good**

```bash
git add dbt-run-results/index.html
git commit -m "chore: verified end-to-end with 200-node test fixture"
```

---

### Task 10: Final polish and production readiness check

**Files:**
- Modify: `dbt-run-results/index.html` (minor polish only)

**Step 1: Verify all checklist items**

- [ ] Page title is `dbt run results — tools.wetzler.net`
- [ ] Back-link to `/` works in header
- [ ] Error message shows on invalid JSON upload
- [ ] Error message shows if `results` array is missing
- [ ] Chart is hidden until a file is loaded
- [ ] Legend is hidden until a file is loaded
- [ ] No data is sent to any server (verify in Network tab — only Google Fonts CDN requests)
- [ ] Works on mobile viewport (bars are still visible, tooltip shows)
- [ ] Homepage card links correctly to `/dbt-run-results/`

**Step 2: Fix any issues found in checklist**

Address each failed item before committing.

**Step 3: Final commit**

```bash
git add dbt-run-results/index.html index.html
git commit -m "feat: complete dbt-run-results visualizer tool"
```
