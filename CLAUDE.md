# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file, zero-dependency drawing app (PowerPoint/OneNote/Paint-style) built with vanilla HTML/CSS/JS. **Everything lives in [paint.html](paint.html)** (~8800 lines): `<style>` (lines 7–673), markup, then one large `<script>` (1206–8837). There is no `package.json`, no build step, no test suite, no npm dependencies.

## Running / developing

- Open [paint.html](paint.html) directly in a modern browser (Chrome/Edge/Firefox). No server needed.
- To iterate: edit the file, reload the browser tab. Changes autosave to IndexedDB, so state persists across reloads.
- To reset all state during debugging, clear the IndexedDB database `paint_html_db` (DevTools → Application → IndexedDB) or use the gallery's delete controls.

## Architecture (all within the `<script>` block)

The script is organized into labeled sections (search for the `/* ====== */` banners). Key ones, in order:

- **App state** (`const app`, ~1413) — the single source of truth. Holds the active drawing record, `app.shapes` (the working array), selection (`selectedIds` Set, `lastHitStack`/`lastHitIndex` for Tab-cycling through overlapping shapes at a pixel), current tool/drag interaction state, canvas config, and `app.style` (per-type default styling that new shapes inherit). Mutating `app.shapes` then calling render + a save is the core loop.
- **Shape model** (~1530) — shapes are plain JSON objects with `id, type, rot, style{...}` plus type-specific geometry. Types: `pen` (points[]), `rect`, `ellipse`, `line`/`arrow` (x1,y1,x2,y2), `text`, `image` (src data URL), `group` (childrenIds[]). Adding a shape type means touching: `shapeDefaults`, the render dispatch, bbox/hit-testing, and the properties panel.
- **Rendering (SVG-native)** (~2013) — shapes render to real SVG elements (root `#svg`), not a raster canvas. The type dispatch is a long `if (s.type === ...)` chain (~2409). SVG-native rendering is a core design choice: it enables crisp export and object-level editing.
- **IndexedDB** (~1353) — DB `paint_html_db` v1, object store `drawings` keyed by `id`, with an `updatedAt` index used to list the gallery most-recent-first. Thin promise wrappers: `idbOpen/idbPut/idbGet/idbDelete/idbList`.
- **History (undo/redo)** (~2891) — `snapshot()`-based per-drawing stack stored on the record itself (`history`, `historyIndex`), so undo state persists in IndexedDB.
- **Autosave + thumbnails** (~2942) — debounced save after edits; regenerates a small PNG thumbnail (offscreen) and writes the whole record. `setSaveState` drives the save indicator.
- **Gallery** (~3282) — left panel listing drawings from `idbList`, with thumbnails; handles switching drawings without losing work.
- **Tools + interaction** (~3550) — `setTool`, pointer handlers, and the drag state machine (`app.dragMode`: draw/move/resize/rotate/marquee).
- **Rich Text Engine** (~4047) — a self-contained `RichTextEngine` (ported from an external `text-by-gemini.html`) powering PowerPoint-style editable text with inline formatting/font-size spans.
- **Alignment guides** (~5746), **Properties panel sync** (~6405, `setRowHidden` + per-type property rows), **Image transparency processing** (~7163), **Export** (~7270, full-res PNG + per-card export popup), **Keyboard shortcuts** (~7789), **UI wiring** (~8351), **Boot** (~8804, `boot()` opens DB and loads/creates the active drawing).

## Conventions specific to this codebase

- **Data shape is a contract**: the persisted drawing record is `{ id, name, shapes, width, height, canvasBg, gridEnabled, thumbnail, updatedAt, history, historyIndex }`. Changing it risks breaking already-saved drawings in users' IndexedDB — prefer additive/optional fields.
- **Per-type style with fallback**: shape style props read as `s.style.<prop> ?? app.style.<prop>` so unset props inherit the global default. Many booleans (flipH/flipV per type, lock-aspect per type) follow this pattern — mirror it when adding style options.
- Utility helpers at the top (~1210): `$`/`$$` selectors, `svgEl`, `clamp`, `uuid`, `now`, `debounce`, color converters (`rgbaFromHexAlpha`, `hexToRgbString`, `rgbStringToHex`), `downloadBlob`, `toast`. Reuse these rather than reimplementing.
- Keyboard shortcuts are central to the UX (arrow keys resize/nudge selection, Option = 1px fine-tune, `t` inserts+edits text, `z`/Shift+`z` undo/redo, Tab cycles overlapping shapes). See [paint_prompt.txt](paint_prompt.txt) for the original spec of intended shortcut/tool behavior.

## Reference

[paint_prompt.txt](paint_prompt.txt) is the original product spec and is the best source for intended behavior/requirements when something is ambiguous.
