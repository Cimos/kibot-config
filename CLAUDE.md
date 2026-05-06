# CLAUDE.md

Guidance for Claude Code working in this repository.

## What this repo is

`kibot-config` is a **GitHub Action plus a library of reusable [KiBot](https://github.com/INTI-CMNB/KiBot) configuration files** for KiCad projects. Downstream KiCad repositories drop in a small workflow file and an `options.yaml`, and CI then produces a full "datapack" of build artifacts (drawings, gerbers, BOM, pick-and-place, 3D STEP models, Blender renders, panel files, visual diffs).

It is published as the GitHub Action `Cimos/kibot-config@main`.

There is no source code to compile and no test suite — every change is exercised by running the action against a real KiCad project. See `docs/memory-map/` for the deep-dive notes.

## Repository layout

```
.
├── action.yaml                  # GitHub Action manifest (Docker action)
├── Dockerfile                   # Active image: kicad9_auto_full:dev
├── Dockerfiles                  # Older/alternative dockerfile (KiCad 8 era), unused at runtime
├── entrypoint.sh                # Container entrypoint — wraps `kibot` CLI
├── local.sh                     # Identical copy of entrypoint for local runs
│
├── options.yaml                 # Default preflight overrides (consumer overrides this)
├── build-pcb-kibot.yaml         # Drawings, BOM, pick-and-place, KiCost, stencils
├── build-3d_model-kibot.yaml    # Simple + full 3D STEP export
├── build-2d_images-kibot.yaml   # 4× Blender renders (top/bottom × angled/straight)
├── 2d_image-kibot.yaml          # Single-render template imported with @VAR@ substitution
├── build-video-kibot.yaml       # Rotating-PCB Blender frame sequence
├── build-diff-kibot.yaml        # KiRi + git-based PCB/SCH diffs
├── build-panel-kibot.yaml       # Drawings for an already-panelised board
├── panelization-base.yaml       # KiKit panelize template (consumer parameterises with @VARS@)
│
├── .github/actions/             # Reference workflow files consumers copy into their repo
│   ├── build-pcb-action.yaml
│   ├── build-cad-action.yaml
│   ├── build-images-action.yaml
│   ├── build-video-action.yaml
│   ├── build-diff-action.yaml
│   └── build-panel-action.yaml
│
└── docs/memory-map/             # In-repo knowledge base — see README there first
```

## Two consumption modes

1. **As a reusable Action.** Downstream repo's workflow does `uses: Cimos/kibot-config@main` and points `config:` at one of the bundled `build-*-kibot.yaml` files (after also `actions/checkout`-ing this repo into `./kibot-config/`).
2. **As a config library.** Consumer writes their own `*.kibot.yaml` and uses `import:` to pull in pieces from this repo (e.g. `panelization-base.yaml` parameterised with project-specific dimensions).

Both modes rely on the consumer providing an `options.yaml` at their repo root that this config tree references via `../options.yaml`.

## Conventions

- All KiBot YAML files start with `kibot: { version: 1 }`.
- Templated variables use KiBot's `@NAME@` substitution (declared under `definitions:` and overridden by `import: { definitions: { ... } }`). This is **not** envsubst.
- Output filename pattern is consistently `'%f-%r-%i.%x'` (filename-revision-id.ext) so artifacts are versioned by KiCad project revision.
- Output directories under `dir:` are short, semantic, and stable (`Drawings`, `BoM`, `Position`, `Model`, `2D_Images`, `Panel`, `DIFF`, `KIRI`).
- Workflows pin `actions/checkout@v4` and `actions/upload-artifact@v4`.
- Workflows derive `PROJECT_NAME` from the first `*.kicad_pro` in the repo root — there is an implicit assumption of a single top-level KiCad project.

## Working in this repo

### Editing KiBot configs

- Reference: <https://kibot.readthedocs.io/>. Output `type:` names (`pdf_sch_print`, `pcb_print`, `position`, `kibom`, `ibom`, `bom`, `kicost`, `step`, `export_3d`, `blender_export`, `panelize`, `kiri`, `diff`, `report`, `stencil_3d`, `stencil_for_jig`) are KiBot-defined.
- Each `*-kibot.yaml` `import:`s `../options.yaml` so that consumer-side overrides (preflight, filters, variants) flow through.
- Per-fab presets (`Elecrow`, `FusionPCB`, `JLCPCB`, `P-Ban`, `PCBWay`) come from KiBot's bundled config library — they are **not** files in this repo.
- The `_KIBOT_F_PASTE` / `_KIBOT_B_PASTE` definitions passed to fab imports control whether paste layers are included in the gerber set.

### Editing the action wrapper

- `entrypoint.sh` and `local.sh` are intended to be **kept identical** — if you edit one, edit the other. (A future refactor on the `test` branch consolidates these; see notes in `docs/memory-map/`.)
- The wrapper accepts sentinel values: `__SCAN__` (let KiBot autodetect), `__NONE__` (omit), `__ALL__` (run every target). These are processed in `args_process` before the inner `kibot` invocation is built.
- `action.yaml` always passes every flag (empty values become no-ops via the sentinel handling). Don't add new inputs without updating both `action.yaml` and `entrypoint.sh`.

### Editing workflows

- The six `.github/actions/*-action.yaml` files are **reference templates** for consumers — they are **not** workflows that run on this repo (they live under `.github/actions/`, not `.github/workflows/`). Treat them as documentation/examples that downstream repos copy.
- Most fire on `push: '**'` and `pull_request → master`. The diff workflow ignores `master` pushes; the video workflow only runs on `master`.
- The `if:` guard on each job (`pull_request.head.repo.full_name != base.repo.full_name`) suppresses double-runs from same-repo PRs. The diff workflow's `if:` has a syntax bug (missing `||` between two clauses) — see gotchas in the memory map.

## Commands

```bash
# Build the action's image locally (matches what GitHub Actions does)
docker build -t kibot-config .

# Dry-run a config against a local KiCad project
docker run --rm -v "$(pwd):/mnt" kibot-config -c kibot-config/build-pcb-kibot.yaml -d output

# Or via local.sh (assumes kibot is on PATH, e.g. inside the upstream container)
./local.sh -c build-pcb-kibot.yaml -b project.kicad_pcb -e project.kicad_sch -d output
```

There are no linters, formatters, or tests configured in this repo.

## Gotchas

See `docs/memory-map/gotchas.md` for the full list. Highlights:

- **`Dockerfiles`** (no extension) is a *separate file* from `Dockerfile`. Only `Dockerfile` is used by the action. Don't edit `Dockerfiles` expecting it to take effect.
- **`local.sh` and `entrypoint.sh` drift.** They are duplicates today — keep them in sync.
- **Bash quoting in entrypoint.** Values are wrapped in single quotes inside variables (e.g. `CONFIG="-c '$VAL'"`) and then re-evaluated via `bash -c`. This means a path containing a single quote will break.
- **`build-panel-kibot.yaml` workflow is two-stage.** The panel action first runs the consumer's `build-panel.yaml` to *generate* the panel `.kicad_pcb`, then runs `build-panel-kibot.yaml` against that generated board. The `board:` input changes between stages.
- **`test` branch is a major refactor in flight** that collapses the six configs into one. If you're asked to work on `main`, don't import its structure.

## Memory map / further reading

The full knowledge base lives in `docs/memory-map/`. Start with `docs/memory-map/README.md`.
