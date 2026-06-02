# paint.html — Self‑Contained SVG Drawing App (IndexedDB Autosave)

A single-file, zero-install drawing app built with plain vanilla HTML/CSS/JavaScript. The goal is a “PowerPoint / OneNote / Paint”-style experience in the browser: draw freehand, place and edit objects (shapes + text), adjust styling (color/stroke/alpha), manage layers, and persist everything locally with automatic saving via IndexedDB.

This repo centers around one file:

- `paint.html` — open it in a browser and start drawing (no build step, no dependencies).

## How to Run
- Download or clone this repo
- Open paint.html in a modern browser (Chrome/Edge/Firefox recommended)
- Draw — changes are saved automatically to your browser’s IndexedDB storage

No server, no npm, no build step.


## Features

### Core drawing + editing
- Freehand drawing with mouse press + move
- Insert objects at cursor:
  - Rectangle
  - Circle
  - Arrow
  - Text (editable like slide editors)
  - (Optionally) additional shapes/tools as implemented
- Object selection by click
- Visual selection bounds (selection rectangle)
- Tab-cycling through overlapping objects at the clicked pixel
- Per-object styling:
  - Stroke size
  - Stroke/fill colors
  - Alpha (opacity)
  - Borders (where applicable)
- Object transforms:
  - Move
  - Resize
  - Rotation
- Paste from clipboard:
  - Screenshots
  - SVG (e.g. shapes taken from svgrepo.com, or other open sources)
- Multi-select and grouping (where implemented)
- Layer management (bring forward/backward)
- Object grouping

### SVG-native rendering
- Shapes are represented as editable JSON and rendered as SVG (not raster canvas), enabling crisp export and object-level editing.

### Autosave + local gallery (IndexedDB)
- Every change is saved automatically (debounced) to IndexedDB
- Multiple drawings supported with a left-side gallery
- Gallery shows thumbnails for fast browsing
- Smooth switching between drawings without losing work

### Undo / Redo (per drawing)
- Per-drawing history stack
- Undo/redo state stored alongside the drawing (IndexedDB)

### Export
- Export the current drawing as full-resolution `.png`