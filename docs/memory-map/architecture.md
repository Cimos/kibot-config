# Architecture

## High-level flow

```
Consumer KiCad repo                     This repo (Cimos/kibot-config@main)
─────────────────────                   ──────────────────────────────────
.github/workflows/build-*.yaml ───────► action.yaml
       │                                       │
       │  uses: Cimos/kibot-config@main         │
       │  with: { config: ..., board: ... }     ▼
       │                                  Dockerfile (FROM kicad9_auto_full:dev)
       │                                       │
       │                                       ▼
       │                                  entrypoint.sh
       │                                       │
       │                                       ▼
       │                                  kibot -c <build-*-kibot.yaml> -b ... -e ...
       │
       └─► options.yaml  ◄── imported by every build-*-kibot.yaml as "../options.yaml"
       └─► panelization.yaml  (optional, parameterises panelization-base.yaml)
       └─► build-panel.yaml   (optional, output of panelization.yaml; stage 1 of panel build)
```

## The two halves

**1. Runtime half (Docker action)**
- `action.yaml` declares inputs and runs the local `Dockerfile`.
- `Dockerfile` extends `ghcr.io/inti-cmnb/kicad9_auto_full:dev` (KiBot + KiCad 9 + KiAuto + KiCost + iBoM + Blender preinstalled).
- `entrypoint.sh` translates the GitHub Action inputs into a `kibot` invocation.

**2. Library half (KiBot YAML configs)**
- Each `build-*-kibot.yaml` declares an `outputs:` list specific to one artifact category.
- All of them `import: ../options.yaml` so consumer-side preflight/filter overrides flow through.
- `2d_image-kibot.yaml` and `panelization-base.yaml` use `@VAR@` substitution to act as templates.

The two halves are decoupled: a consumer can use the action with their own config, or use these configs without the action (running `kibot` directly).

## Why "../options.yaml" and not "options.yaml"

When the action runs, the working directory is the consumer's repo root (mounted at `/mnt`). The kibot-config repo is checked out into `./kibot-config/`. So from a `build-*-kibot.yaml` file's perspective, the consumer's `options.yaml` is one directory up.

This is the contract: **consumers must place `options.yaml` at their repo root.**

## Per-fab presets

The fab-specific imports (`Elecrow`, `FusionPCB`, `JLCPCB`, `P-Ban`, `PCBWay`) referenced in `build-pcb-kibot.yaml` and `build-panel-kibot.yaml` are **bundled with KiBot itself** — they live in the upstream KiBot repository's config library, not here. The `_KIBOT_F_PASTE` / `_KIBOT_B_PASTE` definitions injected at import-time control whether paste layers are excluded from each fab's gerber set (consistently set to exclude paste from gerber output across all five).
