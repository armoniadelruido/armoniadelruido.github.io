---
title: proxmox_backup_collar.sh
date: 2026-07-07 01:00:00
categories: [Sistemas, Scripting]
tags: [bash, proxmox, pbs, backup, pxar]
---

# proxmox_backup_collar.sh

Este script lanza backups con `proxmox-backup-client` de varias rutas origen hacia un repositorio PBS y ejecuta limpieza de memoria en el host remoto al finalizar.

## Ejecucion programada

```cron
30 3 * * * /ruta/origen/scripts/proxmox_backup_collar.sh
```

## Flujo

1. Define log diario.
2. Exporta fichero de password PBS.
3. Exporta fingerprint del servidor PBS.
4. Ejecuta `proxmox-backup-client backup` con varios archivos `.pxar`.
5. Escribe resultado en log.
6. Si termina correctamente, llama por SSH a `liberamemoria.sh` en el host remoto.

## Version saneada

```bash
#!/usr/bin/env bash
set -euo pipefail

LOG="/var/log/proxmox_backup_$(date +%Y%m%d).log"
export PBS_PASSWORD_FILE="/ruta/segura/pbs_password"
export PROXMOX_BACKUP_FINGERPRINT="<FINGERPRINT>"
REPOSITORY="<USUARIO>@pbs@<HOSTNAME>:<DATASTORE>"

proxmox-backup-client backup \
  etc.pxar:/ruta/origen/etc \
  imgs.pxar:/ruta/origen/imagenes \
  descargas.pxar:/ruta/origen/descargas \
  tools.pxar:/ruta/origen/tools \
  multimedia.pxar:/ruta/origen/multimedia \
  site-web.pxar:/ruta/origen/site-web \
  --repository "$REPOSITORY" >> "$LOG" 2>&1

ssh <USUARIO_ADMIN>@<HOSTNAME> "/ruta/origen/scripts/liberamemoria.sh"
```

## Recomendaciones

- Guardar `PBS_PASSWORD_FILE` fuera del repositorio y con permisos `600`.
- Documentar `.pxarexclude` para rutas que no deban respaldarse.
- Probar restauracion de un snapshot periodicamente.
- Evitar SSH como root si puede usarse un usuario dedicado.

<!--
Fuentes consolidadas

- `/tools/scripts/micron1`
- `/tools/scripts/proxmox_backup_collar.sh`
- `/tools/scripts/liberamemoria.sh`
-->
