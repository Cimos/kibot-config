# Glossary

Terms used in this repo's configs, with the minimum context needed to
read the files.

## KiBot output types

| Type              | What it produces                                                         |
| ----------------- | ------------------------------------------------------------------------ |
| `pdf_sch_print`   | Schematic PDF                                                            |
| `pcb_print`       | PCB drawings PDF (multi-page, layer-grouped)                             |
| `position`        | Pick-and-place file (CSV or ASCII)                                       |
| `report`          | Plain-text design report (board stats, layer info, etc.)                 |
| `kibom`           | KiBoM bill of materials (HTML or CSV) — uses field configuration         |
| `ibom`            | Interactive HTML BOM — clickable browser viewer                          |
| `bom`             | KiBot's built-in BOM — separate from KiBoM, supports Digikey/Mouser/LCSC links |
| `kicost`          | KiCost spreadsheet — multi-vendor pricing                                |
| `step`            | Plain STEP 3D model export (KiCad's built-in exporter)                   |
| `export_3d`       | Full 3D export with body/silkscreen/soldermask/zones/cut-vias            |
| `blender_export`  | Calls Blender to render a PCB image (or video frames)                    |
| `panelize`        | KiKit panelization                                                       |
| `kiri`            | KiRi visual git-history web UI                                           |
| `diff`            | Two-revision visual diff (PCB or schematic), generates a PDF             |
| `stencil_3d`      | 3D-printable solder stencil                                              |
| `stencil_for_jig` | Steel stencil + 3D-printed register jig                                  |

## KiBot config keywords

- `kibot.version: 1` — required header on every config file.
- `import:` — pulls in another `.yaml`. Supports `definitions:` to inject `@VAR@` values.
- `outputs:` — list of artifacts to generate.
- `preflight:` — checks/transformations run before outputs (ERC, DRC, zone fills, BOM XML refresh).
- `definitions:` — `@VAR@` substitutions, scoped to the file. Overridden by `import.definitions`.
- `dnf_filter: '_kibom_dnf'` — built-in filter excluding components marked DNF (Do Not Fit) in KiBoM config.
- `_kibom_dnf` — KiBot's bundled DNF-recognition filter.

## Filename templates (`output:` patterns)

| Token | Meaning                                  |
| ----- | ---------------------------------------- |
| `%f`  | Project filename (without extension)     |
| `%r`  | KiCad project revision                   |
| `%i`  | Output's `name:` (used as ID)            |
| `%x`  | File extension (auto-chosen)             |

## KiKit panelization

- **Tabs** — small bridges between boards. `kikit:Tab` is a footprint that marks where to place them.
- **V-cuts** — score lines (no material removal); paired with `User.2` layer here.
- **Backbone** — full-width strip between board rows/columns, used for tooling holes and fiducials.
- **Tooling holes** — non-plated holes for assembly jigs; `3hole` = 3 holes pattern.
- **Fiducials** — copper alignment markers for pick-and-place; `3fid` = 3-fiducial pattern.

## Container / runtime

- `kicad_auto` — INTI-CMNB's all-in-one image: KiCad + KiBot + KiAuto + KiCost + iBoM + KiKit.
- `_full` suffix — adds Blender, Inkscape, ImageMagick.
- `KIBOT_3D_MODELS` — env var pointing to a 3D model cache directory.
- `kibot --quick-start` — auto-generates a minimal config from the project; ignores `-c`/`-b`/`-e`/`-V`.

## Sentinels (this repo's wrapper)

| Sentinel    | Where used        | Effect                                                  |
| ----------- | ----------------- | ------------------------------------------------------- |
| `__SCAN__`  | config/board/sch  | Omit the flag, let kibot autodetect                     |
| `__NONE__`  | targets/variant   | Targets → `-i` (skip outputs); variant → omit           |
| `__ALL__`   | targets           | Run every target                                        |
