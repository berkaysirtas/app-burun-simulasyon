# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A browser-based nose aesthetic simulation tool ("Burun Warp Aracı") — a single-page application that performs interactive mesh-warp image editing entirely in the browser, with no external dependencies.

## Running the App

No build step required. Open `index.html` directly in any modern browser (Chrome 90+, Firefox 88+, Safari 14+, Edge 90+).

## Architecture

The entire application lives in a single `index.html` file with three inline sections:

- **CSS** — layout and UI styling
- **HTML** — upload zone, canvas element, control panel (sliders + buttons)
- **Vanilla JS** — all application logic

### Key subsystems inside the `<script>` block

| Subsystem | Description |
|-----------|-------------|
| Image loading | Reads user-selected file, draws to `<canvas>`, scales to max 800×700px (aspect-ratio preserved) |
| Mesh warp algorithm | Displacement mapping with smooth falloff `(1 - dist²/r²)²`; bilinear interpolation for pixel resampling |
| Undo/redo stack | FIFO queue capped at 20 `Uint8ClampedArray` deep copies |
| Brush preview overlay | Second `pointer-events: none` canvas drawn on top; avoids blocking drag events |
| Download | `canvas.toDataURL()` → PNG |

### Warp math

```
falloff = (1 - dist² / r²)²
srcX    = x - dx × strength × falloff
srcY    = y - dy × strength × falloff
```

Brush parameters: **strength** 5–80, **radius** 20–150. Pixel coordinates are boundary-clamped before access.

### Touch support

`touchstart` / `touchmove` / `touchend` events mirror their mouse counterparts; coordinates are extracted via `touch.clientX/Y`.

## Output

Edited images export as PNG via `canvas.toDataURL('image/png')`. Accepted input formats: JPG, PNG, WEBP.
