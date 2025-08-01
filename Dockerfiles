# FROM ghcr.io/inti-cmnb/kicad9_auto:dev
FROM ghcr.io/inti-cmnb/kicad9_auto:dev_1.8.5-4a1729d_k9.0.1_d_sid

LABEL AUTHOR Madman <madmanuav@icloud.com>
LABEL Description="Export various files from KiCad projects (KiCad 8)"

COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

WORKDIR /mnt

ENTRYPOINT [ "/entrypoint.sh" ]