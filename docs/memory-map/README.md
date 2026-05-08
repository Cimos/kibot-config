# Memory Map

Persistent knowledge base for `kibot-config`. Each file covers one topic.
Keep entries short and load-bearing — facts that are *not* obvious from
reading the code, plus pointers to where the obvious facts live.

## Index

- [`architecture.md`](architecture.md) — How the Action, scripts, and configs fit together.
- [`action-contract.md`](action-contract.md) — The `action.yaml` ↔ `entrypoint.sh` interface, sentinels, and quoting rules.
- [`kibot-configs.md`](kibot-configs.md) — Per-output-config notes (PCB, 3D, images, video, diff, panel) and what each one produces.
- [`panelization.md`](panelization.md) — `panelization-base.yaml` template variables and the two-stage panel build.
- [`workflows.md`](workflows.md) — The six reference GitHub workflows, triggers, and the consumer setup pattern.
- [`docker.md`](docker.md) — Base image, `Dockerfile` vs `Dockerfiles`, and KiCad/KiBot version pinning.
- [`safety-net.md`](safety-net.md) — What automated tests exist (Layer 1 lint), what's planned (Layers 2–4), and why specific tools were rejected.
- [`issues-log.md`](issues-log.md) — Verified bugs (B1–B11) and enhancements (E1–E13), each with file:line evidence and remedy. Mirrored to GitHub issues.
- [`branches-and-history.md`](branches-and-history.md) — `main` vs `test` branch divergence, V1 tag, refactor direction.
- [`gotchas.md`](gotchas.md) — Bugs, footguns, and "don't touch this" notes.
- [`glossary.md`](glossary.md) — KiBot/KiCad/KiKit terms used throughout.

## Conventions for this directory

- One topic per file. If a file grows past ~150 lines, split it.
- Lead each fact with **what** is true; follow with **why** if it's not obvious.
- Mark stale or unverified items with `(unverified)` rather than deleting them.
- Cross-link with relative paths.
