# Branches & History

## Branches

| Branch          | Status        | Notes                                               |
| --------------- | ------------- | --------------------------------------------------- |
| `main`          | active        | What `Cimos/kibot-config@main` resolves to          |
| `test`          | refactor WIP  | Major restructure — see below                       |
| `origin/HEAD`   | → `main`      |                                                     |

## Tags

- `V1` — early snapshot. Not actively used by any action ref.

## `test` branch direction (as of 2026-05)

`git diff main..test --stat` shows a sweeping consolidation:

- **Deletes** the per-category configs: `build-2d_images-kibot.yaml`,
  `build-3d_model-kibot.yaml`, `build-diff-kibot.yaml`,
  `build-panel-kibot.yaml`, `build-video-kibot.yaml`,
  `2d_image-kibot.yaml`.
- **Renames** `build-pcb-kibot.yaml` → `build.kibot.yaml` (single config
  with all outputs).
- **Renames** `panelization-base.yaml` → `build.panelization.base.yaml`.
- **Deletes** `local.sh` (entrypoint becomes the single source of truth).
- **Deletes** `Dockerfiles` (cleanup).
- **Collapses** the six workflow files into one
  `build-datapack-action.yaml` (only the panel workflow is preserved
  separately).
- Net: −770 / +78 lines.

If the user is working on `main`, do not borrow structure from `test` —
the consolidation is incomplete. If they're working on `test`, expect
the configs to be one big file with output filtering instead of separate
files.

## Recent commit themes (last ~30 commits on `main`)

From `git log --oneline`:

- 2025 — incremental output additions: 3D model export refinement, 2D
  image renderer, video action, internal HTML BOM with KiCost links,
  per-vendor stencil/3D options.
- Mid-2025 — diff workflow added, panel build refactored to be
  two-stage, KiBot bumped to 1.8.4 then 1.8.5.
- Earlier — bootstrap, panel work, README.

No conventional-commits prefix is enforced, but most messages start with
`feat:`, `fix:`, `refactor:`, or a bare verb.

## How to check current state

```bash
git -C E:/git/Projects/kibot-config log --oneline -20
git -C E:/git/Projects/kibot-config branch -a
git -C E:/git/Projects/kibot-config diff --stat main test
```
