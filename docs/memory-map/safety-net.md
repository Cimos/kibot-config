# Test Safety Net

What automated checks exist on this repo, why each was chosen, and what's
deferred. Read this before changing the testing setup or claiming "it's
covered."

## Current state (Layer 1 — lint)

`.github/workflows/selftest.yaml` runs on every push (any branch) and on
PRs to `main`. Two jobs:

### `actionlint` job
- Installs upstream actionlint via the published `download-actionlint.bash`.
- Lints `.github/workflows/*.yaml` (real workflows on this repo).
- Lints `.github/actions/*.yaml` (reference templates that consumers copy).
- Embedded shellcheck pinned to `-S error` to match the standalone shellcheck
  job's threshold. Info-level findings are tracked separately
  (e.g. #26 / B11) so they don't drown out genuine regressions.

### `shellcheck` job
- `ludeeus/action-shellcheck@master` against `entrypoint.sh` + `local.sh`.
- `severity: error` (skip warning/info). The known quoting patterns in
  these scripts (SC2089/2090, tracked under E9 / #20) sit at warning
  severity.

## Layers planned but not built

| Layer | Purpose                                          | Blocking on                       |
| ----- | ------------------------------------------------ | --------------------------------- |
| 2     | `kibot --list-targets` parse check on each config | docker image cache wiring         |
| 3     | Smoke run of each `build-*-kibot.yaml` against a fixture | Layer 2 + vendored fixture        |
| 4     | Snapshot hashes of stable text outputs (BoM CSV, position files, design report) | Layer 3 + initial baseline commit |

Layers 3 and 4 require a vendored KiCad project under `tests/fixture/`. By
agreement, this fixture will be hand-authored initially as a placeholder
and replaced with a project-supplied one later.

## Why no yamllint

Tried it and rejected:

- CRLF line endings on every file (Windows save artifact; checked-in as LF
  via `core.autocrlf=true`, so CI doesn't see them — but yamllint does).
- `@VAR@` substitution tokens in `panelization-base.yaml` and
  `2d_image-kibot.yaml` produce **real YAML syntax errors** (the `@`
  character can't start a YAML token). These are intentional — they are
  kibot template markers, substituted before kibot parses the config.
- Indentation/hyphen style nits across nearly every file.

Net: signal-to-noise too low without massive config wrangling. yamllint
is a poor regression detector for *this* repo.

## Why severity=error on both shellcheck instances

The remaining quoting patterns in `entrypoint.sh` (single-quoted variable
strings re-evaluated by `bash -c`) are tracked under E9 (#20) as a
deliberate redesign target. They produce warning-level findings every
time. Surfacing them in CI on every push would mean either:

1. Silencing them per-file with `# shellcheck disable=`, which is harder
   to undo when E9 lands; or
2. Accepting permanent yellow CI, which numbs the team to genuine
   regressions.

Pinning to `severity: error` keeps CI honest as a regression detector.
When E9 is fixed, tighten back to `warning`.

## What "green CI" actually proves

The current setup proves:
- No new YAML syntax errors in any workflow under `.github/workflows/` or `.github/actions/`.
- No new GitHub Actions expression syntax errors.
- No new `severity: error` shellcheck findings in `entrypoint.sh` or `local.sh`.
- No new error-level shellcheck findings in inline `run:` scripts.

It does **not** prove:
- Any `*-kibot.yaml` config still parses correctly.
- The action still produces the outputs it used to.
- 3D / Blender renders still work.
- The docker image still builds.

That's why the bug fixes done so far (B1, B10) were workflow/script-only
— Layer 1 covers their blast radius. Config-touching work (E1, E2, E5,
E6, E11) is on hold until Layer 2 is in place.

## Local-vs-CI gotcha

actionlint v1.7.7 on Windows does **not** bundle shellcheck. CI's actionlint
auto-detects the runner's shellcheck binary and runs it on every inline
`run:` script. So a workflow file that passes `actionlint -color` locally
can still fail CI's actionlint on shellcheck findings.

When this matters, simulate locally with:

```bash
# Strip CRLF (working tree is CRLF; CI sees LF)
tr -d '\r' < entrypoint.sh | shellcheck -S error -

# Or against the staged LF blob:
git show :entrypoint.sh | shellcheck -S error -
```

## Cost

A push to a branch currently triggers ~30s of CI time on a free runner
(both jobs in parallel, both finish in <20s). Layers 2–4 will push this
to ~3–10 minutes per push depending on docker layer cache hit rate.
