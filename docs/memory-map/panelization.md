# Panelization

`panelization-base.yaml` is a **template** for KiKit's `panelize` output. It
is *not* imported by any other config in this repo — consumers `import:` it
from their project-side `panelization.yaml` and override the `@VAR@`
definitions with their board-specific values.

## Two-stage panel build

The panel workflow (`.github/actions/build-panel-action.yaml`) runs in two
stages:

```
Stage 1: build-panel.yaml (consumer-owned, imports panelization-base.yaml)
         ─► generates output_panel/Panel/<project>-panel.kicad_pcb

Stage 2: build-panel-kibot.yaml (this repo)
         ─► runs against the generated panel board to produce drawings, BOM, etc.
```

Stage 1 is gated by `andstor/file-existence-action@v3` checking for
`build-panel.yaml` in the consumer repo. If it's missing, both stages skip.

The two stages pass different `board:` values to the action:
- Stage 1: `${PROJECT_NAME}.kicad_pcb` (the source board)
- Stage 2: `output_panel/Panel/${PROJECT_NAME}-panel.kicad_pcb` (the generated panel)

## Template variables

All panel parameters are KiBot `@VAR@` substitutions.

### Layout

| Variable             | Default       | Meaning                                              |
| -------------------- | ------------- | ---------------------------------------------------- |
| `LAYOUT_SPACE`       | `1`           | mm between PCBs                                      |
| `LAYOUT_H_BACKBONE`  | `3`           | Horizontal backbone width                            |
| `LAYOUT_V_BACKBONE`  | `3`           | Vertical backbone width                              |
| `LAYOUT_ROWS`        | `3`           | Rows of boards                                       |
| `LAYOUT_COLS`        | `3`           | Columns of boards                                    |
| `LAYOUT_ROTATION`    | `0deg`        | Per-board rotation                                   |

### Tabs / Frame

| Variable          | Default      | Meaning                                |
| ----------------- | ------------ | -------------------------------------- |
| `TABS_FOOTPRINT`  | `kikit:Tab`  | Footprint marking tab positions        |
| `FRAME_WIDTH`     | `9`          | Frame width                            |
| `FRAME_SPACE`     | `1`          | Space between frame and boards         |

### Debug

| Variable                      | Default | Meaning                          |
| ----------------------------- | ------- | -------------------------------- |
| `DEBUG_TRACE`                 | `False` | KiKit trace                      |
| `DEBUG_DRAWPARTITIONLINES`    | `False` | Draw partition lines             |
| `DEBUG_DRAWBOXES`             | `False` | Draw bounding boxes              |
| `DEBUG_DRAWTABFAIL`           | `False` | Highlight failed tabs            |
| `DEBUG_DETERMINISTIC`         | `False` | Deterministic mode               |

## Hard-coded panel choices

These are **not** parameterised — change them by editing `panelization-base.yaml`:

- **Cuts**: V-cuts on `User.2` layer, clearance 0.5
- **Copper fill**: Hatched, clearance 0.5, width 0.5, spacing 2
- **Tooling**: 3-hole tooling, 5mm offset
- **Fiducials**: 3-fiducial, 7.5mm offset, 1mm copper, 2mm opening
- **Text**: `{boardTitle}-{boardRevision}` anchored top-centre, 5mm voffset
- **Post**: 0.5mm mill radius, origin bottom-left, dimensions enabled
- **Tabs**: Type `annotation` (footprint-driven, not auto-placed)

## Example consumer `build-panel.yaml`

```yaml
kibot:
  version: 1

import:
  - file: kibot-config/panelization-base.yaml
    definitions:
      LAYOUT_ROWS: 2
      LAYOUT_COLS: 4
      FRAME_WIDTH: 5
      TABS_FOOTPRINT: 'kikit:Tab'
```

(See [Mad_RP2040](https://github.com/Cimos/Mad_RP2040/) for a real example.)
