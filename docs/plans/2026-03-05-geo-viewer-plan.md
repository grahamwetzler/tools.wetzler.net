# Geo Viewer Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a browser-based map viewer at `geo/index.html` that displays geographic data layers (GeoJSON, H3 cells) with auto-coloring, color customization, and hover tooltips.

**Architecture:** Single HTML file with embedded CSS and JS. Leaflet 1.9.x for map rendering with OSM tiles. h3-js via CDN for H3 cell boundary conversion. No framework, no build step.

**Tech Stack:** Vanilla HTML/CSS/JS, Leaflet 1.9.x (CDN), h3-js (CDN)

**Design doc:** `docs/plans/2026-03-05-geo-viewer-design.md`

**Verification:** Since this is a single HTML file with visual output, testing is manual — open `geo/index.html` in a browser after each task. Each task includes specific test data to paste and expected visual results.

---

### Task 1: Scaffold HTML with Leaflet Map and Sidebar Layout

**Files:**
- Create: `geo/index.html`

**Step 1: Create the HTML file with full layout skeleton**

Create `geo/index.html` with:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Geo Viewer</title>
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    html, body { height: 100%; font-family: system-ui, -apple-system, sans-serif; }

    .container { display: flex; height: 100vh; }

    .sidebar {
      width: 300px;
      min-width: 300px;
      background: #1a1a2e;
      color: #e0e0e0;
      display: flex;
      flex-direction: column;
      overflow-y: auto;
    }

    .sidebar h1 {
      font-size: 1.1rem;
      padding: 16px;
      border-bottom: 1px solid #2a2a4a;
      color: #fff;
    }

    .add-layer-section {
      padding: 16px;
      border-bottom: 1px solid #2a2a4a;
    }

    .add-layer-section label {
      display: block;
      font-size: 0.85rem;
      margin-bottom: 6px;
      color: #aaa;
    }

    .add-layer-section select,
    .add-layer-section textarea,
    .add-layer-section button {
      width: 100%;
      font-family: inherit;
    }

    .add-layer-section select {
      padding: 8px;
      background: #2a2a4a;
      color: #e0e0e0;
      border: 1px solid #3a3a5a;
      border-radius: 4px;
      margin-bottom: 12px;
      font-size: 0.9rem;
    }

    .add-layer-section textarea {
      padding: 8px;
      background: #2a2a4a;
      color: #e0e0e0;
      border: 1px solid #3a3a5a;
      border-radius: 4px;
      resize: vertical;
      min-height: 120px;
      margin-bottom: 12px;
      font-size: 0.85rem;
      font-family: 'SF Mono', 'Consolas', 'Monaco', monospace;
    }

    .add-layer-section button {
      padding: 10px;
      background: #4363d8;
      color: #fff;
      border: none;
      border-radius: 4px;
      cursor: pointer;
      font-size: 0.9rem;
      font-weight: 500;
    }

    .add-layer-section button:hover { background: #3453c8; }

    .layers-section {
      padding: 16px;
      flex: 1;
    }

    .layers-section h2 {
      font-size: 0.85rem;
      text-transform: uppercase;
      letter-spacing: 0.05em;
      color: #888;
      margin-bottom: 12px;
    }

    .layer-item {
      display: flex;
      align-items: center;
      gap: 8px;
      padding: 8px;
      border-radius: 4px;
      margin-bottom: 4px;
    }

    .layer-item:hover { background: #2a2a4a; }

    .layer-item .visibility-btn {
      background: none;
      border: none;
      color: #e0e0e0;
      cursor: pointer;
      font-size: 1rem;
      padding: 2px 4px;
    }

    .layer-item .visibility-btn.hidden { opacity: 0.3; }

    .layer-item .layer-name {
      flex: 1;
      font-size: 0.9rem;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
    }

    .layer-item input[type="color"] {
      width: 28px;
      height: 28px;
      border: none;
      border-radius: 4px;
      cursor: pointer;
      background: none;
      padding: 0;
    }

    .layer-item .delete-btn {
      background: none;
      border: none;
      color: #888;
      cursor: pointer;
      font-size: 1rem;
      padding: 2px 6px;
    }

    .layer-item .delete-btn:hover { color: #e6194b; }

    #map { flex: 1; }

    .error-msg {
      color: #e6194b;
      font-size: 0.8rem;
      margin-top: 8px;
    }
  </style>
</head>
<body>
  <div class="container">
    <div class="sidebar">
      <h1>Geo Viewer</h1>
      <div class="add-layer-section">
        <label for="layer-type">Layer type</label>
        <select id="layer-type">
          <option value="geojson">GeoJSON</option>
          <option value="h3">H3 Cells</option>
        </select>
        <label for="layer-data">Data</label>
        <textarea id="layer-data" placeholder="Paste GeoJSON here..."></textarea>
        <button id="add-layer-btn">Add to Map</button>
        <div id="error-msg" class="error-msg"></div>
      </div>
      <div class="layers-section">
        <h2>Layers</h2>
        <div id="layers-list"></div>
      </div>
    </div>
    <div id="map"></div>
  </div>
  <script>
    // App state and map init go here in subsequent tasks
    const map = L.map('map').setView([40, -95], 4);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '&copy; OpenStreetMap contributors',
      maxZoom: 19
    }).addTo(map);
  </script>
</body>
</html>
```

**Step 2: Verify in browser**

Open `geo/index.html` in browser. Expected:
- Dark sidebar on left (300px) with "Geo Viewer" heading, layer type dropdown, textarea, "Add to Map" button, and empty "Layers" section
- Leaflet map fills right side with OSM tiles, centered on US

**Step 3: Commit**

```bash
git add geo/index.html
git commit -m "feat: scaffold geo viewer with Leaflet map and sidebar layout"
```

---

### Task 2: Layer State Management and Color Palette

**Files:**
- Modify: `geo/index.html` (the `<script>` block)

**Step 1: Add layer state and color system**

Add this code inside the `<script>` block, after the tile layer setup:

```javascript
const COLORS = ['#e6194b', '#3cb44b', '#4363d8', '#f58231', '#911eb4',
                '#42d4f4', '#f032e6', '#bfef45', '#fabed4', '#469990'];
let colorIndex = 0;
let layerIdCounter = 0;
const layers = [];

function nextColor() {
  const color = COLORS[colorIndex % COLORS.length];
  colorIndex++;
  return color;
}

function addLayerToState(name, type, color, leafletLayer, data) {
  const layer = {
    id: ++layerIdCounter,
    name,
    type,
    color,
    leafletLayer,
    visible: true,
    data
  };
  layers.push(layer);
  leafletLayer.addTo(map);
  renderLayerList();
  return layer;
}

function removeLayer(id) {
  const idx = layers.findIndex(l => l.id === id);
  if (idx === -1) return;
  map.removeLayer(layers[idx].leafletLayer);
  layers.splice(idx, 1);
  renderLayerList();
}

function toggleLayerVisibility(id) {
  const layer = layers.find(l => l.id === id);
  if (!layer) return;
  layer.visible = !layer.visible;
  if (layer.visible) {
    layer.leafletLayer.addTo(map);
  } else {
    map.removeLayer(layer.leafletLayer);
  }
  renderLayerList();
}

function changeLayerColor(id, newColor) {
  const layer = layers.find(l => l.id === id);
  if (!layer) return;
  layer.color = newColor;
  layer.leafletLayer.setStyle({ color: newColor, fillColor: newColor });
}

function renderLayerList() {
  const container = document.getElementById('layers-list');
  container.innerHTML = '';
  for (const layer of layers) {
    const item = document.createElement('div');
    item.className = 'layer-item';
    item.innerHTML = `
      <button class="visibility-btn ${layer.visible ? '' : 'hidden'}"
              data-id="${layer.id}" title="Toggle visibility">
        ${layer.visible ? '\u{1F441}' : '\u{1F441}'}
      </button>
      <span class="layer-name">${layer.name}</span>
      <input type="color" value="${layer.color}" data-id="${layer.id}" title="Change color">
      <button class="delete-btn" data-id="${layer.id}" title="Remove layer">\u00D7</button>
    `;
    container.appendChild(item);
  }

  container.querySelectorAll('.visibility-btn').forEach(btn => {
    btn.addEventListener('click', () => toggleLayerVisibility(Number(btn.dataset.id)));
  });
  container.querySelectorAll('input[type="color"]').forEach(input => {
    input.addEventListener('input', () => changeLayerColor(Number(input.dataset.id), input.value));
  });
  container.querySelectorAll('.delete-btn').forEach(btn => {
    btn.addEventListener('click', () => removeLayer(Number(btn.dataset.id)));
  });
}
```

**Step 2: Verify in browser**

Open `geo/index.html` — no visual change yet (no layers added), but no console errors. The layer list is empty.

**Step 3: Commit**

```bash
git add geo/index.html
git commit -m "feat: add layer state management and color palette system"
```

---

### Task 3: GeoJSON Layer Support

**Files:**
- Modify: `geo/index.html` (the `<script>` block)

**Step 1: Add GeoJSON parsing and the Add button handler**

Add this code after the layer management functions:

```javascript
let geojsonCount = 0;
let h3Count = 0;

function formatProperties(props) {
  if (!props || Object.keys(props).length === 0) return 'No properties';
  return Object.entries(props)
    .map(([k, v]) => `<b>${k}:</b> ${v}`)
    .join('<br>');
}

function addGeoJSONLayer(text) {
  let geojson;
  try {
    geojson = JSON.parse(text);
  } catch (e) {
    throw new Error('Invalid JSON: ' + e.message);
  }
  if (!geojson.type) {
    throw new Error('Not valid GeoJSON: missing "type" field');
  }

  const color = nextColor();
  geojsonCount++;
  const name = 'GeoJSON ' + geojsonCount;

  const leafletLayer = L.geoJSON(geojson, {
    style: { color, fillColor: color, fillOpacity: 0.3, weight: 2 },
    pointToLayer: (feature, latlng) => L.circleMarker(latlng, {
      radius: 6, color, fillColor: color, fillOpacity: 0.5, weight: 2
    }),
    onEachFeature: (feature, layer) => {
      layer.bindTooltip(formatProperties(feature.properties), { sticky: true });
    }
  });

  const added = addLayerToState(name, 'geojson', color, leafletLayer, geojson);
  map.fitBounds(leafletLayer.getBounds());
  return added;
}

document.getElementById('add-layer-btn').addEventListener('click', () => {
  const type = document.getElementById('layer-type').value;
  const data = document.getElementById('layer-data').value.trim();
  const errorEl = document.getElementById('error-msg');
  errorEl.textContent = '';

  if (!data) {
    errorEl.textContent = 'Please paste some data.';
    return;
  }

  try {
    if (type === 'geojson') {
      addGeoJSONLayer(data);
    } else if (type === 'h3') {
      addH3Layer(data);  // implemented in next task
    }
    document.getElementById('layer-data').value = '';
  } catch (e) {
    errorEl.textContent = e.message;
  }
});
```

**Step 2: Update placeholder text based on dropdown selection**

Add this right after the button event listener:

```javascript
document.getElementById('layer-type').addEventListener('change', (e) => {
  const textarea = document.getElementById('layer-data');
  textarea.placeholder = e.target.value === 'geojson'
    ? 'Paste GeoJSON here...'
    : 'Paste H3 cell IDs (one per line or comma-separated)...';
});
```

**Step 3: Verify in browser**

Open `geo/index.html`, select "GeoJSON", paste this test data, click "Add to Map":

```json
{"type":"FeatureCollection","features":[{"type":"Feature","properties":{"name":"Test Polygon"},"geometry":{"type":"Polygon","coordinates":[[[-104,40],[-104,41],[-103,41],[-103,40],[-104,40]]]}}]}
```

Expected:
- Red polygon appears in Colorado area
- Map zooms to fit the polygon
- Hovering shows tooltip: `name: Test Polygon`
- "GeoJSON 1" appears in sidebar with color picker and delete button
- Clicking delete removes it

**Step 4: Commit**

```bash
git add geo/index.html
git commit -m "feat: add GeoJSON layer parsing, rendering, and hover tooltips"
```

---

### Task 4: H3 Cell Layer Support

**Files:**
- Modify: `geo/index.html` (add h3-js CDN script, add `addH3Layer` function)

**Step 1: Add h3-js CDN script tag**

Add this script tag in the `<head>`, after the Leaflet script:

```html
<script src="https://unpkg.com/h3-js@4"></script>
```

**Step 2: Add the `addH3Layer` function**

Add this function in the `<script>` block, right before the `document.getElementById('add-layer-btn')` event listener:

```javascript
function addH3Layer(text) {
  const ids = text.split(/[\n,]+/).map(s => s.trim()).filter(Boolean);
  if (ids.length === 0) {
    throw new Error('No H3 cell IDs found.');
  }

  // Validate all IDs first
  for (const id of ids) {
    if (!h3.isValidCell(id)) {
      throw new Error(`Invalid H3 cell ID: "${id}"`);
    }
  }

  const color = nextColor();
  h3Count++;
  const name = 'H3 ' + h3Count;

  const polygons = [];
  for (const id of ids) {
    const boundary = h3.cellToBoundary(id);
    // h3.cellToBoundary returns [[lat, lng], ...] — Leaflet needs [lat, lng]
    const latLngs = boundary.map(([lat, lng]) => [lat, lng]);
    const polygon = L.polygon(latLngs, {
      color, fillColor: color, fillOpacity: 0.3, weight: 2
    });
    const res = h3.getResolution(id);
    polygon.bindTooltip(`<b>Cell:</b> ${id}<br><b>Resolution:</b> ${res}`, { sticky: true });
    polygons.push(polygon);
  }

  const layerGroup = L.featureGroup(polygons);
  const added = addLayerToState(name, 'h3', color, layerGroup, ids);
  map.fitBounds(layerGroup.getBounds());
  return added;
}
```

**Step 3: Verify in browser**

Open `geo/index.html`, select "H3 Cells", paste these test IDs, click "Add to Map":

```
8928308280fffff
8928308283fffff
892830828dfffff
```

Expected:
- Three small hexagonal polygons appear (likely in the SF Bay Area at resolution 9)
- Map zooms to fit them
- Hovering shows tooltip with cell ID and resolution
- "H3 1" appears in sidebar layer list
- Color picker works, delete works

**Step 4: Verify both layer types together**

Add a GeoJSON layer, then an H3 layer. They should have different auto-assigned colors. Toggle visibility, change colors, delete — all should work for both.

**Step 5: Commit**

```bash
git add geo/index.html
git commit -m "feat: add H3 cell layer support with boundary rendering and tooltips"
```

---

### Task 5: Polish and Edge Cases

**Files:**
- Modify: `geo/index.html`

**Step 1: Handle `setStyle` for H3 layers (featureGroup of polygons)**

The `changeLayerColor` function calls `setStyle` which works on `L.geoJSON` but also on `L.featureGroup`. Verify this works. If not, update `changeLayerColor`:

```javascript
function changeLayerColor(id, newColor) {
  const layer = layers.find(l => l.id === id);
  if (!layer) return;
  layer.color = newColor;
  if (typeof layer.leafletLayer.setStyle === 'function') {
    layer.leafletLayer.setStyle({ color: newColor, fillColor: newColor });
  } else if (typeof layer.leafletLayer.eachLayer === 'function') {
    layer.leafletLayer.eachLayer(sub => {
      if (typeof sub.setStyle === 'function') {
        sub.setStyle({ color: newColor, fillColor: newColor });
      }
    });
  }
}
```

**Step 2: Add empty-state message in layers list**

In `renderLayerList`, when `layers.length === 0`, show a subtle message:

```javascript
if (layers.length === 0) {
  container.innerHTML = '<div style="color:#666;font-size:0.85rem;">No layers added yet.</div>';
  return;
}
```

**Step 3: Verify everything end-to-end**

1. Open `geo/index.html`
2. Empty state shows "No layers added yet."
3. Add a GeoJSON layer — renders, tooltip works, appears in list
4. Add an H3 layer — renders with different color, tooltip works
5. Toggle visibility on both — layers hide/show
6. Change color on H3 layer — color updates immediately
7. Delete GeoJSON layer — removed from map and list
8. Try invalid JSON — error message shown
9. Try invalid H3 ID — error message shown

**Step 4: Commit**

```bash
git add geo/index.html
git commit -m "feat: polish layer color changes, add empty state, handle edge cases"
```

---

## Summary

| Task | What it builds | Commit |
|------|---------------|--------|
| 1 | HTML scaffold + Leaflet map + sidebar CSS | `feat: scaffold geo viewer...` |
| 2 | Layer state array, color palette, render list, toggle/delete/color | `feat: add layer state management...` |
| 3 | GeoJSON parsing, rendering, hover tooltips, add button | `feat: add GeoJSON layer...` |
| 4 | H3 cell ID parsing, boundary rendering, tooltips | `feat: add H3 cell layer...` |
| 5 | Color change fix for groups, empty state, validation | `feat: polish layer color...` |
