# Workflows

The six files in `.github/actions/` are **reference templates** that consumers copy into their own `.github/workflows/`. They do *not* run on this repo (they live under `.github/actions/`, which GitHub treats as composite-action sources, not workflows).

## Common shape

Every workflow does roughly:

```yaml
on:
  push: { branches: ['**'] }
  pull_request: { branches: [master] }

jobs:
  generate_<thing>:
    runs-on: ubuntu-latest
    if: github.event_name != 'pull_request' || head.repo != base.repo
    steps:
      - actions/checkout@v4 (with submodules: recursive)
      - run: PROJECT_NAME=$(basename *.kicad_pro .kicad_pro) >> $GITHUB_ENV
      - actions/checkout@v4 → ./kibot-config/ from Cimos/kibot-config@main
      - uses: Cimos/kibot-config@main
          with: { config: kibot-config/build-X-kibot.yaml, dir, schema, board, logfile }
      - actions/upload-artifact@v4 (logfile, always())
      - actions/upload-artifact@v4 (datapack)
```

The `if:` guard suppresses double-runs on same-repo PRs (the push event already covers it).

## Per-workflow specifics

| File                       | Job name           | KiBot config                  | Output dir       | Triggers (extras)            |
| -------------------------- | ------------------ | ----------------------------- | ---------------- | ---------------------------- |
| `build-pcb-action.yaml`    | `generate_pcb`     | `build-pcb-kibot.yaml`        | `output_pcb`     | default                      |
| `build-cad-action.yaml`    | `generate_3d_model`| `build-3d_model-kibot.yaml`   | `output_cad`     | default; `verbose: 2`        |
| `build-images-action.yaml` | `generate_2d_images`| `build-2d_images-kibot.yaml` | `output_images`  | default                      |
| `build-video-action.yaml`  | `generate_video`   | `build-video-kibot.yaml`      | `output_video`   | **only on `push: master`**   |
| `build-diff-action.yaml`   | `generate_diff`    | `build-diff-kibot.yaml`       | `output_diff`    | `push`-on-master ignored; `fetch-depth: 0` |
| `build-panel-action.yaml`  | `Panel`            | two-stage (see below)         | `output_panel`   | conditional on `build-panel.yaml` existing |

## Panel workflow's two-stage invocation

```yaml
- uses: andstor/file-existence-action@v3      # check for consumer's build-panel.yaml
  with: { files: "build-panel.yaml" }

- uses: Cimos/kibot-config@main               # stage 1: generate panel pcb
  if: steps.check_files.outputs.files_exists == 'true'
  with:
    config: build-panel.yaml                  # consumer-owned
    board:  ${PROJECT_NAME}.kicad_pcb         # source board

- uses: Cimos/kibot-config@main               # stage 2: build datapack on panel
  if: steps.check_files.outputs.files_exists == 'true'
  with:
    config: kibot-config/build-panel-kibot.yaml
    board:  output_panel/Panel/${PROJECT_NAME}-panel.kicad_pcb
```

## Trigger nuances

- **Diff workflow**: ignores `push` to `master` (no diff against itself), runs on PRs into master, runs on push to other branches. Has `fetch-depth: 0` because the diff config refers to `KIBOT_LAST-10` (10 commits ago), which needs full history.
- **Video workflow**: only runs on `push: master`. Other workflows run on every branch push. Likely a cost/time choice — Blender video frames are expensive.
- **Diff workflow has a malformed `if:`** (see `gotchas.md`) — there are two clauses with no boolean operator joining them.

## Artifact naming

Every artifact name is prefixed with `${PROJECT_NAME}` so multiple
consumers' artifacts coexist if cross-uploaded. Pattern:
- `${PROJECT_NAME}-<kind>-datapack`
- `${PROJECT_NAME}-<kind>-logfile`

(3D model has a typo: `${PROJECT_NAME}--3d_model-logfile` — double dash.)

## Consumer setup

Per the README:

```
SomeProject/
  .github/
    workflows/
      build-pcb-action.yaml      # copied from this repo's .github/actions/
      build-panel-action.yaml    # ditto
      ...
  options.yaml                   # required, overrides default preflights
  build-panel.yaml               # optional, only if you want panelized output
  panelization.yaml              # consumer-named entry point that imports panelization-base.yaml
  SomeProject.kicad_pro
  SomeProject.kicad_sch
  SomeProject.kicad_pcb
```

Note the README says copy to `.github/`, but the active checkouts target
`.github/workflows/` — the workflow files only run there, not under
`.github/actions/` in the consumer repo.
