# KiBot Configs

Each `build-*-kibot.yaml` file declares one category of outputs. They all
`import: ../options.yaml` so consumer overrides flow through.

## `build-pcb-kibot.yaml` — main datapack

Imports per-fab presets (Elecrow, FusionPCB, JLCPCB, P-Ban, PCBWay) — these
files live inside KiBot itself, not here. Each is imported with
`_KIBOT_F_PASTE: '- F.Paste'` and `_KIBOT_B_PASTE: '- B.Paste'` so paste
layers are excluded from gerber output.

Outputs (with `dir:` location):

| Name                 | Type             | dir            | Notes                                                |
| -------------------- | ---------------- | -------------- | ---------------------------------------------------- |
| `print_sch`          | `pdf_sch_print`  | `Drawings`     | Schematic PDF                                        |
| `pcb_pdf`            | `pcb_print`      | `Drawings`     | 4 pages: Front Assembly, Back Assembly, F.Cu, B.Cu   |
| `position (ASCII)`   | `position`       | `Position`     | SMD only, separate front/back, mm                    |
| `position (CSV)`     | `position`       | `Position`     | Same but CSV                                         |
| `report_full`        | `report`         | `Design_Report`| Full design report                                   |
| `stencil_3d`         | `stencil_3d`     | `stencil/3D`   | 3D-printable stencil (thickness 20)                  |
| `stencil_for_jig`    | `stencil_for_jig`| `stencil/Jig`  | Steel stencil + 3D register                          |
| `interactive_bom`    | `ibom`           | `BoM`          | Dark mode, highlight pin1, includes tracks           |
| `bom_html`           | `kibom`          | `BoM`          | KiBoM HTML                                           |
| `bom_csv`            | `kibom`          | `BoM`          | Curated columns: Refs, Value, Rating, Mfr, MPN, Supplier, Supplier PN, Qty, Price |
| `KiCost`             | `kicost`         | `KiCost_kicost`| `output: 'simple'`                                   |
| `bom_internal`       | `bom`            | `KiCost_bom`   | HTML BOM with Digikey/Mouser/LCSC links              |

Preflight `update_xml: true` runs `eeschema_do bom_xml` so the BOM is current.

Filename pattern: `'%f-%r-%i.%x'` everywhere (`%f` = filename, `%r` = revision, `%i` = output id, `%x` = ext).

Commented-out: `KiKit_present_files` — kept as an example for future use.

## `build-3d_model-kibot.yaml` — STEP exports

| Output         | Type        | Notes                                              |
| -------------- | ----------- | -------------------------------------------------- |
| `step_simple`  | `step`      | `dnf_filter: _kibom_dnf`, `no_virtual: true`       |
| `step_full`    | `export_3d` | Full STEP with pads/silkscreen/soldermask/zones, vias cut from body |

`download_lcsc: false` on both — no automatic LCSC model fetching at build time.

## `build-2d_images-kibot.yaml` + `2d_image-kibot.yaml` — Blender renders

`build-2d_images-kibot.yaml` imports `2d_image-kibot.yaml` four times with
different `definitions:` blocks to produce four views into `2D_Images/`:

| Import ID          | View   | rotate_x | rotate_z |
| ------------------ | ------ | -------- | -------- |
| `_top`             | top    | 30       | -20      |
| `_bottom`          | bottom | -30      | -20      |
| `_top_straight`    | top    | 0        | 0        |
| `_bottom_straight` | bottom | 0        | 0        |

`2d_image-kibot.yaml` is the template. Variables it accepts:

- `_KIBOT_IMPORT_ID` — suffix for output name (default `''`)
- `_KIBOT_IMPORT_DIR` — output directory (default `Render_3D`)
- `_KIBOT_3D_VIEW` — `top` / `bottom` / `front` / `back` / `left` / `right` (default `top`)
- `_KIBOT_3D_FILE_ID` — file ID; defaults to `@_KIBOT_IMPORT_ID@`
- `_KIBOT_ROT_X`, `_KIBOT_ROT_Y`, `_KIBOT_ROT_Z` — rotation degrees (default 0)

Render options: `transparent_background: true`, `samples: 10` (low quality, fast).
`pcb3d.download_lcsc: false`.

## `build-video-kibot.yaml` — rotating PCB frames

Single output `3d_video_export` of type `blender_export` with two
points-of-view: a starting pose and a target pose with `steps: 90`. KiBot
interpolates 90 frames between them. Output dir defaults from KiBot.

The video workflow has a comment `# Add step to create MP4 with 2D Frames`
— frame-to-MP4 stitching is not yet implemented.

## `build-diff-kibot.yaml` — visual diffs

| Output     | Type   | Notes                                                                |
| ---------- | ------ | -------------------------------------------------------------------- |
| `kiri`     | `kiri` | KiRi web interface, `keep_generated: true`, `max_commits: 10`        |
| `diff_pcb` | `diff` | PCB diff: `KIBOT_LAST-10` vs `HEAD`, `force_checkout: true`          |
| `diff_sch` | `diff` | Same but `pcb: false` for schematic-only                             |

Requires `fetch-depth: 0` in the workflow checkout (the diff workflow sets this).

The diff config does **not** import `../options.yaml`, unlike the others.
This is intentional — diff runs don't need consumer-side preflight overrides.

## `build-panel-kibot.yaml` — panel-board outputs

Same drawings/BOM-style outputs as the PCB datapack, but configured for an
already-panelized board. Imports the same five fab presets but **without**
the `_KIBOT_F_PASTE` / `_KIBOT_B_PASTE` overrides — paste layers are kept by
default. (Stage 2 of the two-stage panel build; see `panelization.md`.)

## `options.yaml` (default)

Ships with all preflights **off** (`run_erc: false`, `run_drc: false`,
`check_zone_fills: false`). Consumers override this in their own
`options.yaml` — typically to flip ERC/DRC on for production builds.
