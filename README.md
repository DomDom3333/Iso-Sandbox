# Iso Sandbox

A single-file, browser-based scratchpad for checking isometric art against a tile grid.

Drop your PNGs onto the page, tell it what tile size they were drawn for, paint them
onto a grid, and read off the numbers your engine actually needs — tile size, scale,
texture origin, footprint in cells. Nothing is uploaded anywhere; everything runs
locally in the browser — it is a single HTML file you can just double-click.

It is meant as a shared workspace for artists and developers: artists validate their
assets before sending them, developers get the settings the artist used, ready for the
engine.

**▶ [Open Iso Sandbox](https://domdom3333.github.io/Iso-Sandbox)**

---

## Why

Iso Sandbox exists to be a **common interface between artists and developers**, and to cut
down the back-and-forth between them.

Isometric art usually looks wrong the first time it lands in an engine: a tree is two
pixels too tall, a wall does not line up with the floor, a rock is anchored to the wrong
corner. That normally costs a round trip — the artist sends the asset, the developer wires
it up, something is off, and it goes back again.

With Iso Sandbox both sides work from the same page:

- **Artists** can drop their assets onto a real grid and check sizing, anchoring and
  footprint *before* sending anything over — no engine, no build, no developer needed.
- **Developers** get the exact settings the artist landed on — `tile_size`, `scale`,
  `texture_origin`, footprint in cells — as JSON they can feed straight into the engine,
  instead of re-deriving them by eye.

Same file, same numbers, one hand-off instead of five.

## Features

- **Multiple layers**, each with its own tile size and vertical offset — floor, walls,
  and props can sit on different grids at the same time.
- **Ground tiles vs. objects** — ground art fills its cell, objects stand on it and are
  Y-sorted so they can overhang.
- **Anchoring controls** — pick which corner of the covered cells the art sits on, which
  point of the image is pinned there, and a pixel-level fine offset.
- **Multi-cell footprints** up to 24 × 24 cells.
- **Brush** with adjustable area, random coverage (for scattering grass or rocks) and
  random horizontal/vertical mirroring.
- **Reference block** — a plain grey box at the current tile size, for judging whether
  real art is too tall or too wide.
- **Common aspect presets** — 2:1 classic isometric, √3:1 true isometric, 4:1 shallow.
- **Undo/redo**, pan, zoom, grid and focus toggles.
- **JSON export** of every setting and placement, optionally with the images embedded as
  base64 so a scene reopens exactly as you left it.

## Usage

Open the [hosted version](https://domdom3333.github.io/Iso-Sandbox), or grab
`index.html` and **double-click it** — it opens straight in your browser. That is the
whole install. There is no build step, no dependencies and no server required;
`index.html` is the entire application.

```bash
git clone https://github.com/DomDom3333/Iso-Sandbox.git
cd Iso-Sandbox
# double-click index.html, or open it from the browser's File > Open
```

If you would rather serve it over `http://` (handy for sharing it on a LAN, or if your
browser is locked down about `file://` pages), any static server works:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

Then:

1. Drop image files (PNG, JPG, WebP) anywhere on the window.
2. Select an asset and set **Drawn for tile width** to the tile width the art was
   authored against, or press **Fit to cells** to infer it.
3. Choose whether it is a ground tile or an object, how many cells it covers, and where
   it sits.
4. Drag on the canvas to paint. Hold <kbd>Alt</kbd> and drag to erase.
5. Open the **Scene** section to save or copy the JSON.

### Shortcuts

| Key | Action |
| --- | --- |
| drag | Place art |
| <kbd>Alt</kbd> + drag | Erase |
| right-drag | Pan |
| wheel | Zoom |
| <kbd>1</kbd>…<kbd>9</kbd> | Pick art |
| <kbd>Shift</kbd> + <kbd>1</kbd>…<kbd>9</kbd> | Pick layer |
| <kbd>-</kbd> / <kbd>=</kbd> | Brush area |
| <kbd>[</kbd> / <kbd>]</kbd> | Size of selected art |
| <kbd>Ctrl</kbd> + <kbd>Z</kbd> / <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>Z</kbd> | Undo / redo |
| <kbd>G</kbd> / <kbd>F</kbd> | Toggle grid / focus |
| <kbd>?</kbd> | Show the shortcut list |

> **Note:** nothing is stored between sessions. Save a scene JSON before closing the tab.

## Scene format

`Save JSON file` (or `Copy`) produces a document like this:

```jsonc
{
  "format": "iso-sandbox",
  "version": 1,
  "exported": "2026-01-01T00:00:00.000Z",
  "conventions": { "cell": "...", "texture_origin": "...", "note": "..." },
  "tiles": [
    {
      "id": 0,
      "name": "grass.png",
      "source_file": "grass.png",
      "type": "tile",                          // "tile" (ground) or "sprite" (object)
      "texture_size":        { "x": 256, "y": 128 },
      "authored_tile_size":  { "x": 128, "y": 64 },
      "size_in_tiles":       { "x": 1, "y": 1 },
      "scale": 1.0,
      "texture_origin":      { "x": 0, "y": 0 },
      "image_data": "data:image/png;base64,…", // only when "Include the images" is on
      "editor": { "anchor": "front", "image_pivot": "bottom",
                  "origin_offset": { "x": 0, "y": 0 }, "cursor": "center" }
    }
  ],
  "layers": [
    {
      "name": "Ground",
      "visible": true,
      "tile_size": { "x": 128, "y": 64 },
      "y_offset": 0,
      "cells": [ { "tile": 0, "x": 0, "y": 0, "flip_h": false, "flip_v": false } ]
    }
  ]
}
```

Conventions:

- `x`/`y` on a cell are TileMap cell coordinates; `+x` runs down-right on screen and
  `+y` down-left.
- `texture_origin` is measured in pixels in the tile's own space, from a centred-on-cell
  draw; `+y` moves the art **up**. It uses the `tile_size` aspect of the first layer the
  tile is placed on.
- The `editor` block is Iso Sandbox's own state. Engines can ignore it; it exists so a
  saved scene reopens with the same controls.

Exports **without** embedded images still load — the scene opens with placeholders, and
dropping the matching art back in relinks it by filename. You can also drop a `.json`
straight onto the canvas to open it.

## Browser support

Any current desktop browser (Chrome, Firefox, Edge, Safari). It needs Canvas 2D, the
File API and drag-and-drop. Not designed for touch or small screens.

## Contributing

Bug reports, ideas and pull requests are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md).

## License

[GNU Affero General Public License v3.0](LICENSE).
