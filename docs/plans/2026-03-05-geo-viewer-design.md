# Geo Viewer Tool Design

**URL:** `tools.wetzler.net/geo`
**Date:** 2026-03-05

## Overview

A browser-based map viewer that lets users add and visualize geographic data layers. Supports GeoJSON and H3 grid cells with automatic coloring, color customization, and hover details.

## Architecture

Single HTML file (`geo/index.html`) with embedded CSS and JS. No build step, no framework. Libraries via CDN:

- **Leaflet 1.9.x** — map rendering, OpenStreetMap tiles
- **h3-js** — H3 cell ID to polygon boundary conversion

## Layout

```
+------ Sidebar (300px) ------+-------- Map (rest) --------+
| Layer type: [GeoJSON v]     |                             |
| +-------------------------+ |                             |
| | Paste data here...      | |       Leaflet Map           |
| +-------------------------+ |       (OSM tiles)           |
| [Add to Map]                |                             |
|                             |                             |
| --- Layers ---              |                             |
| [eye] Layer 1  [color] [x] |                             |
| [eye] Layer 2  [color] [x] |                             |
+-----------------------------+-----------------------------+
```

- Sidebar: fixed 300px left panel
- Map: fills remaining viewport
- Responsive: sidebar collapses or overlays on small screens (stretch goal)

## Layer Management

- Layers stored as JS array: `{ id, name, type, color, leafletLayer, visible, data }`
- Auto-assigned colors from palette, cycling through 10 distinct colors
- Each layer in sidebar shows: visibility toggle, name, color picker (`<input type="color">`), delete button
- Auto-naming: "GeoJSON 1", "H3 2", etc.

## Layer Types

### GeoJSON
- User pastes JSON into textarea
- Validate as GeoJSON (check for `type` field)
- Render via `L.geoJSON()` with layer color as fill/stroke

### H3 Cells
- User pastes H3 cell IDs (newline or comma-separated)
- Convert each ID to polygon via `h3.cellToBoundary()`
- Render as Leaflet polygons with layer color

### Future Types
- Architecture supports adding new types via a layer type dropdown and corresponding parse/render functions

## Hover Behavior

- Leaflet tooltip on mouseover showing all feature properties as key-value pairs
- H3 cells show: cell ID and resolution
- GeoJSON features show: all properties from the feature

## Color System

Palette: `['#e6194b', '#3cb44b', '#4363d8', '#f58231', '#911eb4', '#42d4f4', '#f032e6', '#bfef45', '#fabed4', '#469990']`

- New layers pick next color in sequence
- User overrides via native color picker
- Color changes update Leaflet layer style in real-time

## File Structure

```
geo/
  index.html    (single file: HTML + CSS + JS)
```

## Tech Decisions

- No framework — vanilla HTML/CSS/JS with modern features (ES modules not needed, CDN scripts)
- No build step — single file served as-is
- Leaflet chosen for simplicity and native GeoJSON support
- h3-js loaded via CDN for H3 cell boundary computation
