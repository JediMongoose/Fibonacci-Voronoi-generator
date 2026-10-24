# Fibonacci Voronoi plate generator

A single-file web app that generates a round, 3D-printable plate cut with one hole per cell of a Voronoi diagram built on a Fibonacci ("sunflower") spiral. It is designed for a **backlit LED panel** made of separate printed parts:

| Part | Typical filament | Purpose |
|---|---|---|
| Walls and border | black | Opaque Voronoi web that separates each LED |
| Diffuser cells | white / translucent | Fills each hole so the light is diffused |
| LED mask | any | Disc with one LED hole per cell, pockets for the LED modules and optional debossed wiring lines |
| Wiring guide (optional) | any | Raised ribs showing how to chain the LEDs in sequence |

The design was reverse-engineered from `Voroni_fibonacci.stl` (172 mm disc, 2 mm thick, 256 cells, 0.8 mm walls) and extended for large panels, multi-colour printing and splitting into printable quarters.

## Files

| File | What it is |
|---|---|
| `voronoi_fibonacci_app.html` | The web app. Self-contained: no build step, no server needed. |
| `voronoi_fibonacci.scad` | The original OpenSCAD script for the plate only (86 mm radius). It does **not** include any of the later features (mask, diffuser, wiring, split, 3MF). |

## Running it

Open `voronoi_fibonacci_app.html` in a modern browser (Chrome, Edge, Firefox, Safari). It also works hosted on any static web server.

- Everything runs in the browser. The two libraries it needs (earcut for triangulation, polygon-clipping for 2D booleans) are embedded in the file, so it works offline. Only the web font is loaded from Google Fonts, with a system fallback.
- The preview updates as you move any slider.

## How the pattern works

- Seed *n* sits at radius `k * sqrt(n + 0.5)` and angle `225° - n * 137.5077°` (the golden angle).
- The spiral scale `k` is set automatically so the pattern fills the plate: `k = 0.9542 * (R - border + wall/2) / sqrt(N - 0.5)`.
- Each hole is the Voronoi cell of its seed, inset by half the wall width, and clipped to a circle of radius `plate radius - border width`.
- With the original settings (R = 86 mm, 256 cells, 5.6 mm border, 0.8 mm walls) the result matches the source STL to within about 0.025 mm per cell.

## Settings

| Setting | Range | Default | Notes |
|---|---|---|---|
| Plate radius | 40 – 600 mm | 260 | |
| Wall height | 0.4 – 100 mm | 2 | Height of the thin walls between cells |
| Border height | 0.4 – 100 mm | 2 | Height of the solid outer rim, separate from the walls |
| Number of cells | 20 – 800 | 256 | |
| Wall between holes | 0.3 – 15 mm | 0.8 | |
| Border width | 1 – 150 mm | 5.6 | Solid rim between the outermost holes and the plate edge |
| LED mask hole diameter | 0.5 – 40 mm | 5 | Capped to about 95% of the largest circle that fits in a cell |
| LED mask thickness | 0.4 – 10 mm | 2 | |
| LED mask size clearance | -5 – 10 mm | 1 | Shrinks the mask's outer edge (negative grows it) |
| LED module pocket diameter | 0 – 30 mm | 10 | 0 turns pockets off. Capped to about 97% of the cell |
| LED module pocket depth | 0.2 – 10 mm | 2 | On the bottom of the mask. Capped to leave 0.2 mm of mask |
| Diffuser height | 0.2 – 100 mm | 2 | |
| Diffuser clearance | -2 – 1 mm | 0.1 | Shrinks each cell. Negative values make the diffuser overlap the walls |
| Debossed line width / depth | 0.3 – 5 mm / 0.1 – 5 mm | 1 / 0.6 | Only used when "Deboss wiring lines on LED mask" is ticked |
| Wiring guide rib width / height | 0.6 – 10 mm / 0.5 – 15 mm | 2 / 3 | For the separate raised guide |
| Wiring segments (spiral arms) | 5, 8, 13, 21, 34 | 13 | |

Checkboxes: **Show wiring guide in preview**, **Deboss wiring lines on LED mask**, **Split into 4 quarters for printing**.

## Wiring segments

The LEDs are chained along the spiral arms of the Fibonacci pattern: seeds `r`, `r+M`, `r+2M`, ... for an arm count `M`. Each arm is its own segment, drawn as a line from hole edge to hole edge. There is no connecting line between segments.

- Green dot = first LED of a segment (innermost), red dot = last (outermost). These markers are preview-only.
- The line under the preview reports how many LEDs each segment has. If they differ, it suggests a cell count that makes them all equal (for example, 260 cells with 13 arms gives 20 LEDs per segment). Where segments differ by one, wire two LEDs as one to compensate.

## Splitting into quarters

Tick **Split into 4 quarters for printing** to cut the plate into four pieces for printers with a small bed.

- Each cell is assigned to a quarter by the angle of its seed (0–90°, 90–180°, ...). The cut follows the **middle of the walls**, so it never passes through a hole.
- Each side of a cut keeps half a wall. With 0.8 mm walls that is 0.4 mm per piece, so a thicker wall makes sturdier edges to glue.
- Because the cut zig-zags around whole cells, a quarter is a little larger than `R x R`, by up to about one cell width. A higher cell count reduces the overshoot.
- The magenta lines in the preview show the cuts.
- The walls, diffuser cells and LED mask are all split along the same lines, so quarter 1 of each fits together.

## Downloads

| Button | Result |
|---|---|
| Download STL | The walls and border. With the split on: a zip of 4 quarters |
| Download LED mask STL | The mask. With the split on: a zip of 4 quarters |
| Download wiring guide STL | The raised ribs, one solid per LED-to-LED section |
| Download diffuser cells STL | One solid per cell. With the split on: a zip of 4 quarters |
| Download 3MF of all parts | See below |
| Download SVG outline | 2D outline at true size (mm), useful for laser cutting |
| Save / Load settings to file | JSON settings file |
| Reset settings to defaults | Restores the defaults and clears the saved browser settings |

Files in zips have the current date and time.

### LED mask details

The mask is a disc with one round hole per LED, placed at the centre of each cell. Two optional features are cut into its **bottom face**:

- **Pockets** around each LED hole for the LED modules (default 10 mm diameter, 2 mm deep).
- **Debossed wiring lines** along the wiring segments, connecting the LED holes.

The mask is built from stacked layers, so the STL contains a hidden internal face between layers. Slicers merge this without trouble.

### 3MF export (Snapmaker U1)

The 3MF is laid out for a Snapmaker U1 (270 x 270 x 270 mm bed, four toolheads, sliced with Snapmaker Orca):

- With the split on: plates 1–4 each hold one wall quarter plus its matching diffuser quarter, grouped together in their assembled position. The LED mask quarters follow on their own plate(s), packed onto as few plates as fit.
- With the split off: plate 1 holds the walls and diffuser, plate 2 the mask.
- Walls, border and mask are assigned to **extruder 1 (black)** and the diffuser to **extruder 2 (white)**. Map these to your filament slots in the slicer.
- The raised wiring guide is **not** included in the 3MF. Download it separately if you want it.
- Parts that are too large for the 270 mm bed are listed under the buttons. Turn on the split or reduce the plate radius. As a guide, a plate radius of about 230–240 mm with 256 cells keeps the quarters near the bed size.

The plate layout follows Orca's conventions but has not been tested on a printer or in Snapmaker Orca. If the plates or colours do not carry over, use "Arrange all" and assign the filaments by hand.

## Settings storage

Settings are saved automatically to the browser's local storage, with a cookie as a fallback, and restored when you reopen the page. Use "Save settings to file" for a portable copy and "Load settings from file" to restore it. The cookie may not persist in some views, such as pages opened straight from disk.

## Running inside claude.ai

If the page is viewed as a Claude artifact, the viewer only allows certain file types to be saved. STL and 3MF files are therefore delivered as `.zip` files (rename `voronoi_fibonacci_3mf.zip` to `.3mf`). Opened locally, you get plain `.stl` and `.3mf` files.

## Limitations

- Not tested on a physical print or in Snapmaker Orca.
- The raised wiring guide is a single piece and is not split into quarters.
- At the highest cell counts the preview and exports take a few seconds.
- `voronoi_fibonacci.scad` reproduces the original plate only.

## Third-party code

Embedded in the HTML file:

- [earcut](https://github.com/mapbox/earcut) 2.2.4 (ISC licence): polygon triangulation
- [polygon-clipping](https://github.com/mfogel/polygon-clipping) 0.15.7 (MIT licence): 2D union, difference and intersection

No licence has been chosen for the app itself. Add one before sharing it.
