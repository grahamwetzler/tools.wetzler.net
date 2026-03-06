# dbt Run Results Visualizer — Design

**URL:** `tools.wetzler.net/dbt-run-results/`
**Date:** 2026-03-06

## Overview

A browser-based Gantt chart visualizer for dbt `run_results.json` files. Users upload the artifact and see a thread-by-thread timeline of node execution, with color coding by node type and status, bottleneck highlighting, and hover tooltips. Everything runs client-side — no data leaves the browser.

## Architecture

Single file: `dbt-run-results/index.html` with inline CSS and JS. No dependencies, no build step. Rendering via HTML5 Canvas. Matches existing project conventions.

## Data Model

From `run_results.json`, extract per result:

| Field | Source | Notes |
|---|---|---|
| `node_name` | `unique_id` | Strip `model.`, `test.`, etc. prefix |
| `node_type` | `unique_id` prefix | `model`, `test`, `snapshot`, `seed`, `source`, `analysis` |
| `status` | `status` | `success`, `error`, `warn`, `skip`, `pass`, `fail` |
| `thread_id` | `thread_id` | Determines chart row |
| `started_at` | `min(timing[].started_at)` | Bar x-offset origin |
| `completed_at` | `max(timing[].completed_at)` | Bar x-offset end |
| `execution_time` | `execution_time` | Tooltip display |
| `compile_time` | `timing[name=compile]` duration | Tooltip breakdown |
| `execute_time` | `timing[name=execute]` duration | Tooltip breakdown |

**Layout:** `run_start = min(all started_at)`. Bar x = `(started_at - run_start) / total_duration * canvas_width`. Bar width = `execution_time / total_duration * canvas_width`.

## Visual Design

### Color System

Node type sets the base color (dark theme palette):

| Type | Color |
|---|---|
| `model` | `#7c9fff` (blue) |
| `test` | `#6fcf97` (green) |
| `snapshot` | `#bb8fce` (purple) |
| `seed` | `#e6a96a` (orange) |
| `source` | `#5bb8c4` (teal) |
| `analysis` | `#f0d080` (yellow) |

Status modifies the base color:
- `success` / `pass` — full opacity base color
- `error` / `fail` — red tint overlay + bright red top border
- `warn` — yellow top border
- `skip` — 30% opacity + hatched fill pattern

### Bottleneck Highlighting

Two complementary signals:
1. **Statistical outlier** — nodes in the top 10% of execution time get a `⚡` glyph at bar left edge and slightly brighter fill
2. **Critical thread** — the thread whose last node finishes latest gets its row label rendered in `var(--accent)` with a `★` marker (this thread determined total run duration)

No DAG required — both signals derive purely from timing data.

### Canvas Layout

```
┌─────────────────────────────────────────────────────┐
│  dbt run results          [Upload run_results.json] │
├──────────┬──────────────────────────────────────────┤
│ thread-1 │ ██████░░░░████░░░░░░░░░░░░░░░░░░░░░░░░  │
│ thread-2 │ ░░░░░░███░░░░░████████░░░░░░░░░░░░░░░░  │
│ thread-3 │ ░░░░░░░░░░░░░░░░░░░░░░░░░░░██████░░░░░  │
├──────────┴──────────────────────────────────────────┤
│  0s              15s              30s          45s  │
└─────────────────────────────────────────────────────┘
```

- Thread label column: 80px fixed
- Row height: 32px with 4px gap
- Time axis: relative elapsed time from run start, pinned to bottom
- Canvas scrolls vertically for many threads; axis stays visible

### Legend

Inline legend below canvas: color swatches for each node type + status modifier examples.

## Interactions

### File Upload
- Drag-and-drop zone + click-to-browse button
- Parsed client-side via `JSON.parse` + `FileReader`
- Inline error message on parse failure or missing required fields

### Tooltip (hover)
- `mousemove` hit-tests against stored array of bar rects
- Floating `<div>` positioned near cursor (not canvas overlay)
- Contents: node name, type badge, status badge, execution time, compile/execute split, `⚡ bottleneck` if applicable

## Homepage

Add a tool card to `index.html` for `dbt-run-results/` alongside the geo viewer.

## Non-Goals

- No manifest.json integration (no DAG-based critical path)
- No server-side processing
- No export/share functionality
- No WebAssembly (Canvas + JS handles 2,000+ nodes without measurable lag)
