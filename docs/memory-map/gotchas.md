# Gotchas

Bugs, footguns, and "this is intentional, don't 'fix' it" notes.

## Real bugs

### `build-diff-action.yaml` `if:` is malformed
Lines ~14-17:
```yaml
if: github.event_name != 'pull_request' ||
  github.event.pull_request.head.repo.full_name != github.event.pull_request.base.repo.full_name
  github.ref != 'refs/heads/master' ||
  github.ref == 'refs/heads/main'
```
There's a missing `||` (or `&&`) between line 2 and line 3. GitHub Actions parses this loosely and the expression likely evaluates true on most events, but the intent is unclear.

**Don't blindly fix it without confirming intent** — the right joiner depends on whether the check is "skip on master pushes" (already covered by `branches-ignore: master`) or something else.

### 3D model logfile artifact has a double dash
`build-cad-action.yaml`:
```yaml
name: ${{ env.PROJECT_NAME }}--3d_model-logfile
```
Cosmetic only; produces e.g. `MyBoard--3d_model-logfile.zip`.

### `LICENSE.md` has placeholders
Reads `Copyright (c) 2024,` (no holder) and `Neither the name of [project]`. Effectively unenforceable as written. Don't touch unless the user asks — this is the author's call.

## Intentional duplication / quirks

### `entrypoint.sh` ↔ `local.sh` are duplicates
By design (see `action-contract.md`). `local.sh` is for running outside
Docker. **Edit both or neither.**

### `Dockerfiles` (no extension) is a parallel scratch dockerfile
**Not used at runtime.** See `docker.md`. Don't assume edits to it take effect.

### `action.yaml` always passes every flag
Even when the user doesn't supply an input, `action.yaml` evaluates
`${{ inputs.X }}` to its default and passes it. The sentinels
(`__SCAN__`, `__NONE__`, `__ALL__`) are how empty values get normalized
inside `entrypoint.sh`. Don't try to make flags conditional in
`action.yaml`.

### `build-diff-kibot.yaml` does **not** import `../options.yaml`
The other configs do; diff doesn't need consumer-side preflights. Likely
intentional, not a missed line.

### Single-quote wrapping inside variables
`entrypoint.sh` builds `CONFIG="-c '$VAL'"` and runs the result through
`bash -c`. Paths with embedded single-quotes will break. Spaces are fine.

### `additional_args` is unquoted
Passed raw into `bash -c`. The user is expected to supply a fully-formed
shell fragment. Don't add quoting — it's the escape hatch.

## "Don't trust the README" notes

### README says copy `.github/` folder
The README's setup instructions tell consumers to "Add `.github` folder",
but the workflow files in `.github/actions/` need to land in
`.github/workflows/` to actually run. Strictly copying `.github/` produces
a broken setup.

### README says project layout has `workflows/` at root
Same section is malformed — shows `workflows/` once at the top level and
once nested under `.github/`. Only the nested location is real.

## Things that look broken but aren't

### `update_xml: true` only in `build-pcb-kibot.yaml`
Other configs don't need a fresh BOM XML, so they don't run it. Don't
duplicate it.

### `download_lcsc: false` everywhere
This repo doesn't fetch LCSC 3D models at build time (would slow CI and
fail on rate-limits). If a consumer wants LCSC models, they install them
locally and use `install3D: YES` / `cache3D: YES`.

### Panel workflow's `build-panel.yaml` reference
Stage 1 calls `build-panel.yaml` (consumer-owned), not anything in this
repo. The consumer authors that file by `import:`-ing
`panelization-base.yaml` from this repo and overriding the `@VAR@`
definitions.

## Open / unverified

- **Workflow trigger uses `master`, not `main`.** The PR triggers and the
  diff filter both refer to `master`. The repo's default branch is `main`.
  Either the templates expect consumer repos to use `master`, or these are
  wrong and should be `main`. (unverified — confirm with the author before
  changing.)

- **README's V1 tag** doesn't appear to be referenced anywhere. Consumers
  pin `@main`, not `@V1`.

- **`additional_args` shell injection.** It's an action input, so a
  malicious workflow caller could inject anything. Mitigated by the action
  running in a Docker sandbox per-job, but worth knowing.
