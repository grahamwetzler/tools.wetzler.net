# Geo Layer Controls Design

**Date:** 2026-03-13

## Overview

Add three layer management features to the geo viewer: rename, opacity control, and drag-to-reorder. Remove the existing visibility toggle.

## Layer Row Layout

Each row (left → right):

```
[≡ drag] [name / inline input] [✏ rename] [● color+opacity] [✕ delete]
```

All icons are inline SVG, 16×16, `stroke="currentColor"`, 1.5px stroke-width (Heroicons outline style). The existing eye emoji visibility toggle is removed entirely.

## Feature: Rename (Pencil Icon)

- Clicking the pencil SVG icon turns the name `<span>` into an `<input>` in place
- Enter or blur commits the new name and saves to `localStorage`
- Escape cancels and restores the original name

## Feature: Color + Opacity Popover

- Replaces `<input type="color">` with a styled button (color swatch)
- Clicking opens a small absolute-positioned popover containing:
  - A native `<input type="color">` for hue selection
  - A horizontal range slider for opacity (0–100%)
  - A numeric label showing current opacity %
- Clicking outside the popover closes it
- Layer state gains an `opacity` field (default 1.0), persisted to `localStorage`
- Applies to Leaflet layer via `setStyle({ opacity, fillOpacity: opacity * 0.5 })`

## Feature: Drag-to-Reorder (Hamburger Handle)

- `≡` handle uses `cursor: grab` and `draggable="true"` on the row
- HTML5 `dragstart` / `dragover` / `drop` events reorder the `layers` array
- After drop, Leaflet z-order is updated: layers are iterated in order and brought to front sequentially so the last in the list is on top
- `localStorage` is saved after each reorder

## Data Model Changes

- Add `opacity: number` (0–1) to each layer entry in state and `localStorage`
- Remove `visible: boolean` from state and `localStorage`
- Remove `toggleLayerVisibility` function

## Approach

Option A: HTML5 drag-and-drop (zero additional dependencies). Chosen for simplicity and consistency with the project's existing CDN-only pattern.
