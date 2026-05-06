# Issues & Enhancements Log

Verified findings from a second-pass audit of the repo (2026-05-06).
Each entry: severity, evidence (file:line), proposed remedy. Anything I
could not directly verify is marked `(unverified)`.

## Bugs

### B1 — `build-diff-action.yaml` `if:` is missing an operator
**Severity:** medium (would fail expression eval if/when this template is copied to a real workflow).

`.github/actions/build-diff-action.yaml:14-17`:
```yaml
if: github.event_name != 'pull_request' ||
  github.event.pull_request.head.repo.full_name != github.event.pull_request.base.repo.full_name
  github.ref != 'refs/heads/master' ||
  github.ref == 'refs/heads/main'
```
Between line 15 and line 16 there's no `||` / `&&`. GitHub Actions parses `if:` as a single expression, so this is invalid syntax.

**Remedy:** Decide intent (skip on master pushes? something else?) then add the right joiner. Currently dormant because the file lives in `.github/actions/`, not `.github/workflows/` — but breaks for any consumer who copies it.

### B2 — 3D-model logfile artifact has a double dash
**Severity:** low (cosmetic).

`.github/actions/build-cad-action.yaml:45`:
```yaml
name: ${{ env.PROJECT_NAME }}--3d_model-logfile
```
Produces e.g. `MyBoard--3d_model-logfile.zip`.

**Remedy:** Single dash to match the other workflows.

### B3 — `LICENSE.md` has unfilled placeholders
**Severity:** medium (legal clarity).

- `LICENSE.md:2` — `Copyright (c) 2024,` (no holder name).
- `LICENSE.md:15` — `Neither the name of [project]`.

**Remedy:** Fill in name (Cimos / MadMan) and project name.

### B4 — README project-layout tree is malformed
**Severity:** medium (misleads consumers).

`README.md:30-43`:
```
SomeProject/
  .github/
  ...
  .github/
    workflows/
  workflows/
    build-datapack.yaml
    build-panel.yaml
```
`.github/` appears twice; a stray top-level `workflows/` exists; the nesting is broken.

**Remedy:** Replace with a correct tree showing files at consumer-repo paths.

### B5 — README references `build-datapack.yaml` which doesn't exist on `main`
**Severity:** medium.

`README.md:42` lists `build-datapack.yaml`. Actual workflow files on `main` are `build-{pcb,cad,images,video,diff,panel}-action.yaml`. The name `build-datapack.yaml` matches the consolidation on the `test` branch.

**Remedy:** Either reflect the actual file names on `main`, or land the `test` branch first and update accordingly.

### B6 — Panel workflow stage-1 logfile is never uploaded
**Severity:** medium.

`.github/actions/build-panel-action.yaml`:
- Stage 1 (line 46) writes to `logfile/logfile-panel.log`.
- Stage 2 (line 56) writes to `logfile_panel/logfile.log`.
- Upload step (line 63) only uploads `path: logfile_panel`.

So if stage 1 fails, its log is lost.

**Remedy:** Use the same logfile dir for both stages, or upload both.

### B7 — Workflows mix `master` (PR triggers) and `main` (kibot-config ref)
**Severity:** medium (silently skips PRs on `main`-default repos).

Every workflow under `.github/actions/` triggers on `pull_request: { branches: [master] }` but checks out `Cimos/kibot-config@main`. If a consumer's default branch is `main` (modern default), PRs into `main` won't trigger anything.

**Remedy:** Either make the trigger branch configurable (input) or document the assumption / update to `main`.

### B8 — `entrypoint.sh` and `local.sh` are byte-identical duplicates
**Severity:** low (drift risk).

SHA256 `9533fb7c...` matches; both 201 lines / 5614 bytes. There's no automated check enforcing equality.

**Remedy:** Have `local.sh` `exec entrypoint.sh "$@"`, or symlink, or delete `local.sh` (the `test` branch already deletes it).

### B9 — Panel workflow upload step lacks the `files_exists` guard
**Severity:** low–medium (depends on `upload-artifact@v4` behaviour for missing path).

`.github/actions/build-panel-action.yaml:65-69` "Upload Panel Results" has no `if: steps.check_files.outputs.files_exists == 'true'` guard, while the build steps do. On consumers without `build-panel.yaml`, the upload step still runs and may fail.

**Remedy:** Add the same `if:` guard.

### B10 — `entrypoint.sh:37` illegal-option message mishandles `$@`
**Severity:** low (cosmetic; affects error message formatting).

`entrypoint.sh:37` and `local.sh:37`:

```bash
echo -e "$SCRIPT: illegal option $@"
```

shellcheck SC2145: mixes string and array. `echo` takes one string but `$@` expands as an array. Should be `$*` (joins with `$IFS` space) when the intent is a single concatenated message.

**Found by:** shellcheck `severity: error` in the new self-test workflow (the first regression the safety net surfaces beyond the known-bug list).

**Remedy:** `$@` → `$*`.

## Enhancements

### E1 — Inconsistent `output:` filename patterns
**Where:** `build-pcb-kibot.yaml`.

Most outputs use `output: '%f-%r-%i.%x'`. Outliers:
- `report_full`, `stencil_3d`, `stencil_for_jig`, `KiCost` — no `output:` set (KiBot defaults).
- `bom_internal` — `output: '%f.%x'` (no rev/id).
- `interactive_bom` — `name_format: '%f_%r_iBoM'` (underscores) plus `output: '%f-%r-%i.%x'`.

**Remedy:** Pick one convention and apply everywhere.

### E2 — `KiCost` output dirs are clunky (`KiCost_kicost`, `KiCost_bom`)
**Where:** `build-pcb-kibot.yaml:169, 177`.

Looks like a mid-development naming. Could be `KiCost/spreadsheet/` and `KiCost/bom/` or similar.

### E3 — `Dockerfiles` (no extension) is dead/unused
**Where:** repo root.

Parallel scratch dockerfile, not referenced by `action.yaml`. Source of confusion. The `test` branch already deletes it.

**Remedy:** Delete on `main`, or rename to e.g. `Dockerfile.pinned` and document its purpose.

### E4 — README "Future work" is partially stale
**Where:** `README.md:46-51`.

- "Add support for variants" — variants are already supported via the `variant` action input.
- "Add 2d render generation" — done (`build-2d_images-kibot.yaml`).
- "Add GIF generation" — still missing.
- "Default action shim" — unclear.

**Remedy:** Update list.

### E5 — `build-diff-kibot.yaml` does not import `../options.yaml`
**Where:** `build-diff-kibot.yaml:1-4`.

The other five build configs import options. Diff doesn't. **Possibly intentional** (diffs don't need preflight overrides), but worth a comment in the file or memory map.

**Remedy:** Add a brief comment in the file confirming intent, or import for consistency.

### E6 — Commented-out `KiKit_present_files` block
**Where:** `build-pcb-kibot.yaml:111-116`.

Dead comment block. Either uncomment with a real config or delete.

### E7 — `.github/actions/` reference workflows are easy to misread as composite actions
**Where:** the whole `.github/actions/` directory.

GitHub treats `.github/actions/` as a location for composite actions. These files are full workflow definitions (with `on:`, `jobs:`), not composite actions. Consumers can only use them by *copying* into their `.github/workflows/`. The path is misleading.

**Remedy:** Move them under `examples/workflows/` or `templates/` and update README accordingly. (The `test` branch keeps them in `.github/actions/`, so this would be a divergent improvement.)

### E8 — No self-test / smoke run for the action
**Where:** repo root, missing.

There is no CI on this repo that runs the action against a known KiCad project. Every change is exercised manually downstream.

**Remedy:** Add a `.github/workflows/selftest.yaml` that builds the docker image and runs `build-pcb-kibot.yaml` against a tiny vendored test project. Catches regressions before they reach `Mad_RP2040`-style consumers.

### E9 — `entrypoint.sh` quoting is fragile around single quotes
**Where:** `entrypoint.sh:99-148`.

Values are wrapped as `"-c '$VAL'"` and re-evaluated via `bash -c`. Paths containing single quotes break.

**Remedy:** Use an array (`KIBOT_ARGS+=(-c "$VAL")`) and `kibot "${KIBOT_ARGS[@]}"` instead of `bash -c "$STRING"`. (`additional_args` is the only one that genuinely wants string concatenation.)

### E10 — `additional_args` (`-x`) shell-injection risk
**Severity:** documentation gap, not a real vulnerability.

The action input `additional_args` is concatenated unquoted into `bash -c`. A malicious workflow caller could inject anything — but they could already do so via any other action. Worth documenting that this input is **raw shell**.

### E11 — Fab preset paste-layer behaviour is undocumented
**Where:** `build-pcb-kibot.yaml:7-29`, `build-panel-kibot.yaml:7-11`.

`build-pcb-kibot.yaml` overrides `_KIBOT_F_PASTE` / `_KIBOT_B_PASTE` with `'- F.Paste'` / `'- B.Paste'`. `build-panel-kibot.yaml` does not. The semantic difference (whether paste layers end up in fab gerbers) is not documented in either file. Consumers reading this can't tell why.

**Remedy:** One-line comment in each file explaining the intent. (unverified — confirms what the override actually does in the upstream KiBot preset.)

### E12 — Consumer setup is not pinnable
**Where:** all reference workflows.

Every workflow uses `Cimos/kibot-config@main`. If `main` breaks, every consumer breaks at once. Action versioning (release tags + Major-version float, e.g. `@v1`) would let consumers pin and upgrade deliberately.

**Remedy:** Tag releases (`v1`, `v1.0`, `v1.0.0`), update reference workflows to `@v1`, document upgrade path.

### E13 — No GIF generation (carryover from README "Future work")
Stub for tracking.

## Out-of-scope / known by author

- License is permissive 3-clause BSD, intentional. Don't propose changing the license model.
- `test` branch is an active in-flight refactor; not part of this issue triage.
