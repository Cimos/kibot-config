# Docker

## Active image

`Dockerfile` (the one `action.yaml` uses):

```dockerfile
FROM ghcr.io/inti-cmnb/kicad9_auto_full:dev
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
WORKDIR /mnt
ENTRYPOINT [ "/entrypoint.sh" ]
```

Base: `kicad9_auto_full:dev` from [INTI-CMNB/KiBot](https://github.com/INTI-CMNB/KiBot). The `_full` suffix means it includes the heavyweight optional deps:
- KiCad 9.x
- KiBot (latest dev)
- KiAuto (`pcbnew_do`, `eeschema_do`)
- KiCost
- iBoM (Interactive HTML BOM)
- KiKit (panelization)
- Blender (for `blender_export` outputs — renders, video frames)
- Inkscape, ImageMagick (for image conversion)

The `:dev` tag is mutable; CI grabs whatever is current at run time.

`WORKDIR /mnt` matches GitHub Actions' default mount point for the consumer repo.

## Inactive `Dockerfiles`

The file named `Dockerfiles` (no extension) is **not** used by `action.yaml`.
It pins an older image:

```dockerfile
# FROM ghcr.io/inti-cmnb/kicad9_auto:dev
FROM ghcr.io/inti-cmnb/kicad9_auto:dev_1.8.5-4a1729d_k9.0.1_d_sid
LABEL Description="Export various files from KiCad projects (KiCad 8)"
```

This appears to be a stash for pinning to a known-good image (KiBot 1.8.5,
KiCad 9.0.1) when `:dev` breaks. The label still says "KiCad 8", inherited
from before the bump.

**Treat `Dockerfiles` as scratch.** If you need a known-good build, edit
`Dockerfile` to point at a pinned tag rather than maintaining a parallel
file.

## What's preinstalled (run-time banner)

`entrypoint.sh` prints versions before doing work:

```
KiBot: <version>
KiCad: <version>
Debian: <release>
KiAuto: <version>
KiCost: <version>
iBoM: <version>
```

If a CI run misbehaves, the logfile starts with this banner — useful for
pinning the failure to a base image change.

## Local build

```bash
docker build -t kibot-config .
docker run --rm -v "$(pwd):/mnt" kibot-config -c kibot-config/build-pcb-kibot.yaml -d output
```

`-v "$(pwd):/mnt"` matches the GitHub Actions mount; `kibot` finds the
KiCad files automatically (or via `__SCAN__`).
