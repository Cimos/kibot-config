# Action Contract

The contract between `action.yaml` (GitHub Action interface) and
`entrypoint.sh` (container CLI).

## Inputs (`action.yaml`)

| Input             | Default      | Flag in entrypoint | Purpose                                         |
| ----------------- | ------------ | ------------------ | ----------------------------------------------- |
| `config`          | `__SCAN__`   | `-c`               | Path to a `*.kibot.yaml` config file            |
| `dir`             | `output`     | `-d`               | Output directory prefix                         |
| `board`           | `__SCAN__`   | `-b`               | `.kicad_pcb` path                               |
| `schema`          | `__SCAN__`   | `-e`               | `.kicad_sch` path                               |
| `targets`         | `__ALL__`    | `-t`               | Space-separated target list                     |
| `variant`         | `__NONE__`   | `-V`               | KiBot global variant                            |
| `install3D`       | `NO`         | `-i`               | Run `kicad_3d_install.sh` before kibot          |
| `cache3D`         | `NO`         | `-C`               | Set `KIBOT_3D_MODELS=$HOME/cache_3d`            |
| `quickstart`      | `NO`         | `-q`               | Use `kibot --quick-start` (ignores other flags) |
| `verbose`         | `0`          | `-v`               | 0–4, mapped to ``/`-v`/`-vv`/`-vvv`/`-vvvv`     |
| `logfile`         | `''`         | `-L`               | Pass-through to kibot `-L`                      |
| `additional_args` | `''`         | `-x`               | Raw extra args concatenated onto kibot cmdline  |

`action.yaml` *always* passes every flag (e.g. `-V ${{ inputs.variant }}`).
Empty/sentinel values are normalised to no-ops by `entrypoint.sh`.

## Sentinel values

| Sentinel    | Meaning                                                  |
| ----------- | -------------------------------------------------------- |
| `__SCAN__`  | Omit the flag entirely; let kibot autodiscover the file  |
| `__NONE__`  | For `targets`: pass `-i` (no targets, run preflights only). For `variant`: omit. |
| `__ALL__`   | For `targets`: omit (run all targets)                    |

## Quoting rules

`entrypoint.sh` builds each flag as a string with **embedded single quotes**:

```bash
CONFIG="-c '$VAL'"
```

…and then re-evaluates the whole thing through `bash -c`:

```bash
/bin/bash -c "kibot $CONFIG $DIR $BOARD $SCHEMA $VERBOSE $VARIANT $TARGETS $LOG_FILE $EXTRA_ARGS"
```

**Implications:**

- A path containing a single quote (`'`) will break (rare, but worth knowing).
- `additional_args` is **not** quoted — it's expected to be a fully-formed shell fragment.
- Spaces in paths are handled by the embedded single quotes.

## Pre-run setup (in `run` function)

Before invoking `kibot`, `entrypoint.sh`:

1. Runs `kicad-git-filters.py` if `.git/` exists (sets up KiCad-specific git filters for cleaner diffs).
2. Runs `kicad_3d_install.sh` if `install3D=YES`.
3. Exports `KIBOT_3D_MODELS=$HOME/cache_3d` if `cache3D=YES`.
4. Prints version banner: KiBot, KiCad, Debian, KiAuto, KiCost, iBoM versions (via top-of-file commands run before `main`).

If `quickstart=YES`, none of the other flags are used — kibot runs `--quick-start` to autodiscover everything.

## `entrypoint.sh` ↔ `local.sh`

These two files are **byte-for-byte duplicates** (same SHA, same shebang, same logic). `entrypoint.sh` is what the container runs; `local.sh` is intended for running the same script outside Docker (assumes `kibot` is on PATH).

**Rule: edit both, or neither.** If they ever diverge on `main`, that's a bug. (The `test` branch deletes `local.sh` entirely and rewrites the entrypoint — see `branches-and-history.md`.)
