# Wandering-Tiles-Generative-Art

A seed-based generative art catalogue for **Wandering Tiles**, a computational line-drawing system. Built as a single-file HTML page with p5.js, styled as a gallery-ready portfolio and licensing surface.

**Live:** `https://reyrove.github.io/Wandering-Tiles-Generative-Art/`

---

## Overview

**Wandering Tiles** renders a 4–5 × 4–5 grid of tiles from numeric seeds. Inside each tile, a walker takes a fixed number of steps. At every step, its direction is updated by a simple parity rule: even grid positions turn one way, odd positions turn the other. A containment check flips the walker 180° when it approaches the tile's margin — so the walk never leaves its tile.

The result is a grid of meandering lines, each a single continuous gesture drawn in p5's `curveVertex` with no fill, on a white ground.

The page functions as a **catalogue and licensing surface** for fashion houses, textile studios, and surface designers. Every composition is reproducible by seed, meaning the same number always produces the identical walk.

---

## Features

### Generative Engine (p5.js)

- **4–5 × 4–5 tile grid** — 16–25 tiles per composition, each with its own wandering curve
- **Parity-rule walk** — direction updated at every step by grid-position parity:
  ```js
  if ((gx + gy) % 2 === 0) dir = (dir + turnBias) % 4;
  else                     dir = (dir - turnBias) % 4;
  ```
- **Turn bias** — +1 or −1 per seed, setting the handedness of the walk
- **Containment check** — direction flips 180° at a 12% margin inset, keeping the walk inside its tile
- **Single continuous curve** — drawn with `curveVertex` and `endShape()` — no fill, no breaks
- **Black lines on white ground** — a classic sketch character, with stroke weight scaled to canvas (`cs × 0.0018`)
- **Per-tile randomness** — step size (`tileSize × 0.045–0.055`) and starting direction drawn per tile from the same seed

### Same Seed → Many Surfaces

A single seed drives every instance on the page:

| Instance | Purpose | Tiling |
|----------|---------|--------|
| **Cover** | Hero composition | 1 × 1 |
| **Plate** | Main work (master) | 1 × 1 |
| **Frame** | Framed archival print | 1 × 1 |
| **Print** | Archival print surface | 1 × 1 |
| **Scarf** | Silk scarf (3:1) | 3 × 1 |
| **Textile** | Fabric yardage (4:3) | 2 × 2 |
| **Wall** | Wallpaper (2:3) | 1 × 2 |
| **Archive** | 8 unique seeds | 1 × 1 each |

### Controls

- **New Seed** button / `R` key — regenerate with a fresh seed
- **Download** button / `S` key — export the plate canvas as PNG
- **Click any archive plate** — load that seed into the main view

### Performance

- Debounced resize handling (250ms)
- `IntersectionObserver` pauses off-screen sketches
- `visibilitychange` pauses rendering when the tab is hidden
- `pixelDensity(1)` for a lighter canvas footprint
- Two-frame render (`maxFrames: 2`) so tiles draw once and stop

---

## File Structure

```
Wandering-Tiles-Generative-Art/
├── index.html          # Single-file page (structure + styles + generative engine)
├── README.md
└── images/
    ├── fav.svg         # Favicon
    ├── tote.png        # Mockup: tote bag
    ├── tee.png         # Mockup: t-shirt
    └── cushion.png     # Mockup: cushion
```

No build step. No dependencies to install. p5.js is loaded from a CDN.

---

## Getting Started

1. Clone the repo:
   ```bash
   git clone https://github.com/reyrove/Wandering-Tiles-Generative-Art.git
   cd Wandering-Tiles-Generative-Art
   ```
2. Add the three mockup images to `images/` (optional — they appear in the Commission section).
3. Open `index.html` in a browser.

That's it. The page self-initialises on `DOMContentLoaded`.

---

## How the Generative System Works

### Seeded Random

A linear congruential generator replaces p5's native randomness inside each sketch:

```js
class SeededRandom {
  constructor(s) { this.state = (Math.abs(s) % 2147483647) || 1; }
  rand() { this.state = (this.state * 9301 + 49297) % 233280; return this.state / 233280; }
  range(min, max) { return this.rand() * (max - min) + min; }
  int(min, max) { return Math.floor(this.range(min, max + 1)); }
  pick(arr) { return arr[Math.floor(this.rand() * arr.length)]; }
}
```

Every aesthetic decision — tile count, turn bias, starting direction, step size — is drawn from this generator in a fixed order. Same seed, same sequence, same composition.

### Parameter Space (per seed)

| Parameter | Range |
|-----------|-------|
| Tile count | 4 × 4 to 5 × 5 |
| Turn bias | +1 (clockwise) or −1 (counter) |
| Step size (per tile) | `tileSize × 0.045 – 0.055` |
| Steps per tile | `floor(tileSize × 0.75)` |
| Starting direction (per tile) | 0 / 1 / 2 / 3 (N / E / S / W) |
| Margin | 12% of tile size |

### The Parity Walk

Each tile contains a single walk. The walker starts at the tile centre, in a random cardinal direction. At each step:

```js
const gx = floor((wx - ox) / stepSize);
const gy = floor((wy - oy) / stepSize);

// Parity rule: even positions turn one way, odd positions the other
if ((gx + gy) % 2 === 0) {
  dir = (dir + turnBias + 4) % 4;
} else {
  dir = (dir - turnBias + 4) % 4;
}

wx += dirs[dir][0] * stepSize;
wy += dirs[dir][1] * stepSize;

// Containment: flip 180° at the margin
if (wx < ox + margin || wx > ox + tileSize - margin ||
    wy < oy + margin || wy > oy + tileSize - margin) {
  dir = (dir + 2) % 4;
}

curveVertex(wx, wy);
```

The parity rule creates a meandering walk that changes direction at a fixed spatial frequency. The containment check makes sure it never escapes its tile — even as it wanders near the edges.

### The Curve

The walk is drawn with p5's `curveVertex` and `endShape()` — no fill, no explicit stroke besides the black default. Each vertex is a point the walker visited in sequence. The result is a smooth, continuous line that reads as a single gesture, even though it was built step by step.

### Tiling Logic

Rectangular surfaces (scarf, textile, wall) tile the square composition N×M. Each tile is drawn into its own translated coordinate space, centred within its cell:

```js
const tileW = canvasW / tileCols;
const tileH = canvasH / tileRows;
const tileSize = Math.min(tileW, tileH);
// draw square of side tileSize at (ox, oy) for each cell
```

This preserves the square composition's proportions regardless of surface aspect ratio.

---

## Customisation

### Change the default seed

Edit the init function:

```js
function init() {
  seed = 123456789;  // fixed seed instead of random
  // ...
}
```

### Adjust the tile count range

Edit the `sr.int(4, 5)` call inside `drawComposition`:

```js
const tileCount = sr.int(4, 5);  // change to sr.int(3, 6) for 3–6 tiles
```

Larger grids produce finer, denser webs. Smaller grids produce chunkier, more individual walks.

### Adjust the walk density

Two knobs inside `drawComposition`:

- **Step size:** `tileSize * sr.range(0.045, 0.055)` — smaller steps produce finer webs.
- **Steps per tile:** `Math.floor(tileSize * 0.75)` — more steps produce denser walks that fill the tile more completely.

### Adjust the margin

```js
const margin = tileSize * 0.12;
```

A larger margin keeps the walk further from the tile edge — more breathing room but less fill. A smaller margin lets the walk crowd the boundary more tightly.

### Change the stroke weight

```js
p.strokeWeight(cs * 0.0018);
```

The current ratio (0.18% of canvas) reads as a fine pen. Increase for a heavier ink, decrease for a hairline.

### Change the stroke colour

```js
p.stroke(0, 0, 0);
```

Replace with any HSB triple. To make the lines a warm charcoal instead of pure black:

```js
p.stroke(0, 0, 20);
```

### Change the background

```js
p.background(0, 0, 100);
```

Replace with any HSB triple. To make a warm paper instead of pure white:

```js
p.background(40, 5, 98);
```

### Change surface tiling

Edit `surfaceDefs`:

```js
const surfaceDefs = [
  { id: 'surfacePrint',   wrapId: 'wrap-surfacePrint',   tile: { cols: 1, rows: 1 } },
  { id: 'surfaceScarf',   wrapId: 'wrap-surfaceScarf',   tile: { cols: 3, rows: 1 } },
  // ...
];
```

The `tile` object accepts any `{ cols, rows }` pair.

### Replace mockup images

Swap the files in `images/` or update the `src` attributes in the Commission section:

```html
<img src="images/your-mockup.png" alt="..." loading="lazy" />
```

---

## Design System

| Token | Value | Use |
|-------|-------|-----|
| `--bg` | `#0a0a0a` | Page background |
| `--paper` | `#f4f1ea` | Light surface (Surfaces section) |
| `--gold` | `#b8935a` | Primary accent |
| `--gold-soft` | `#d4b483` | Italic emphasis |
| `--serif` | Cormorant Garamond | Headings, titles |
| `--mono` | DM Mono | Labels, metadata, UI |

Type scale, spacing, and section rhythm match the Skyline, Portrait, Girih, Gothic Grid, Order to Chaos, and Wallpaper Groups templates. Responsive breakpoints at 900px, 720px, 560px, and 400px.

---

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `R` | Regenerate with a new seed |
| `S` | Download plate as PNG |
| `Enter` / `Space` | Activate focused archive item |
| `Esc` | Close legal modal |

---

## Browser Support

Modern evergreen browsers with:

- Canvas 2D (universal)
- `IntersectionObserver` (universal in modern browsers)
- `backdrop-filter` (Safari 9+, Chrome 76+, Firefox 103+)
- CSS custom properties (universal)

Fallback: if `IntersectionObserver` is unavailable, sketches render continuously.

---

## Accessibility

- Archive items are keyboard-focusable with `role="button"` and `aria-label`
- Legal modal traps focus, closes on `Esc` and overlay click
- `prefers-reduced-motion` disables transitions and smooth scroll
- All interactive elements have visible focus states (via browser default)

---

## Section Map

| Section | ID | Purpose |
|---------|-----|---------|
| Cover | — | Hero title + live composition |
| Statement | `#statement` | Artist statement |
| Plate | `#work` | Main work + metadata + controls |
| Surfaces | `#application` | Framed print + 4 tiled surfaces |
| Process | `#process` | Three process cards |
| Archive | `#archive` | 8 selectable seeds |
| Commission | `#commission` | Services + mockups + CTA |
| Colophon | — | Footer, links, legal |
| Legal modal | `#legalModal` | Licensing / Terms / Credits |

---

## Deployment (GitHub Pages)

1. Push the repo to GitHub.
2. Go to **Settings → Pages**.
3. Under **Source**, select `Deploy from a branch`.
4. Choose branch `main` and folder `/ (root)`.
5. Save — the site goes live at `https://reyrove.github.io/Wandering-Tiles-Generative-Art/`.

---

## Notes on the Source Sketch

The generative engine is adapted almost verbatim from the original p5 sketch. The adaptations that turn it into a catalogue page:

1. **Seeded random replaces p5's `random()`** — a local `SeededRandom` class drives every choice. The original sketch calls `setSeeds()` for fxhash compatibility; that's replaced with the seeded generator directly.
2. **Sketch factory** — `makeSketch({ slotId, seedValue, ... })` returns a p5 sketch that can be instantiated into any DOM slot, enabling cover / plate / frame / surfaces / archive from one seed.
3. **DOM-driven canvas sizing** — `cs = min(windowHeight, windowWidth)` becomes slot-rect dimensions.
4. **Tiling support** — the `tile: { cols, rows }` parameter repeats the square composition across rectangular surfaces.
5. **Drawing logic takes `p` explicitly** — the original used p5's global mode; here `p` is passed so the factory closure works.
6. **`int(random(...))` and `floor(random(...))` replaced with `sr.int(...)`** — fully deterministic per seed.
7. **`curveVertex` preserved** — the smooth meander is central to the character of the piece, so this p5 primitive is kept.
8. **`windowResized` replaced by a debounced `handleResize`** — all sketches are destroyed and recreated on resize (250ms debounce), since their canvases are locked to slot dimensions.
9. **`triggerPreview` removed** — the original calls it for fxhash thumbnail generation; not needed in the plain-browser catalogue.

---

## Related Repositories

| Repo | Theme |
|------|-------|
| [`Skyline-Series`](https://github.com/reyrove/Skyline-Series) | Themed cityscapes |
| [`Portrait-Series-Generative-Art`](https://github.com/reyrove/Portrait-Series-Generative-Art) | Generative faces |
| [`Girih-2-Generative-Art`](https://github.com/reyrove/Girih-2-Generative-Art) | Islamic geometric ornament |
| [`Gothic-Grid-Generative-Art`](https://github.com/reyrove/Gothic-Grid-Generative-Art) | Mosaic grid composition |
| [`Order-to-Chaos-Generative-Art`](https://github.com/reyrove/Order-to-Chaos-Generative-Art) | Gradient grid composition |
| [`Wallpaper-Groups-Generative-Art`](https://github.com/reyrove/Wallpaper-Groups-Generative-Art) | Crystallographic symmetry |
| **`Wandering-Tiles-Generative-Art`** | Line-drawing composition *(this repo)* |

All seven share the same catalogue template, design tokens, and seed-based reproducibility model.

---

## Credits

- **Design & Generative System** — Reyhaneh Daneshdoost
- **Typefaces** — Cormorant Garamond, DM Mono
- **Engine** — p5.js 1.9.0
- **Edition** — Wandering Tiles, Autumn 2026

---

## License

All artwork, code, and generative systems are the intellectual property of the artist. Compositions may not be reproduced, redistributed, resold, or adapted without a written license.

For licensing, commissions, or private systems: **reyhanehdaneshdoost@gmail.com**