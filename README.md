# kibot-config

A GitHub Action plus a library of [KiBot](https://github.com/INTI-CMNB/KiBot) configs for building complete KiCad datapacks &mdash; drawings, gerbers, BOM, pick-and-place, 3D STEP, Blender renders, panels, and visual diffs &mdash; on every push.

## Quick start

In your KiCad project repo, drop a workflow at `.github/workflows/build-pcb.yaml`:

```yaml
name: Build PCB datapack
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: recursive
      - uses: actions/checkout@v4
        with:
          repository: Cimos/kibot-config
          path: kibot-config
      - uses: Cimos/kibot-config@main
        with:
          config: kibot-config/build-pcb-kibot.yaml
      - uses: actions/upload-artifact@v4
        with:
          name: pcb-datapack
          path: output/
```

Add an `options.yaml` at the root of your repo with your KiBot preflight overrides &mdash; see [Cimos/Mad_RP2040](https://github.com/Cimos/Mad_RP2040/) for a working example.

## Project layout it expects

```text
your-kicad-project/
├── your-project.kicad_pro
├── your-project.kicad_sch
├── your-project.kicad_pcb
├── options.yaml              # required — preflight, filters, variants
├── panelization.yaml         # optional — only if using build-panel
└── .github/
    └── workflows/
        └── build-pcb.yaml
```

The action derives the project name from the first `*.kicad_pro` it finds at the repo root, so there's an implicit assumption of a single top-level KiCad project.

## Bundled configs

Point the action's `config:` input at one of these:

| Config | What it builds |
|---|---|
| `build-pcb-kibot.yaml` | Drawings (schematic + PCB PDFs), gerbers per fab preset, BOM, pick-and-place, KiCost, stencils |
| `build-3d_model-kibot.yaml` | 3D STEP export (simple + full) |
| `build-2d_images-kibot.yaml` | 4× Blender renders (top/bottom × angled/straight) |
| `build-video-kibot.yaml` | Rotating-PCB Blender frame sequence |
| `build-diff-kibot.yaml` | KiRi + git-based PCB/SCH visual diffs |
| `build-panel-kibot.yaml` | Drawings for an already-panelised board |

Reference workflow templates for each live in [`.github/actions/`](.github/actions) of this repo &mdash; copy them into your own repo's `.github/workflows/` to wire up multi-job builds.

## Example

[Cimos/Mad_RP2040](https://github.com/Cimos/Mad_RP2040/) is a real KiCad project using kibot-config end-to-end. Browse its `.github/workflows/` and `options.yaml` to see how the pieces fit together.

## License

I've chosen a permissive license so the community can use this freely. If you build something with it, I encourage you to open-source your project in return.
