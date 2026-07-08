---
title: sync_files_a_nextcloud.sh
date: 2026-07-07 00:50:00
categories: [Sistemas, Scripting]
tags: [bash, rsync, nextcloud, nfs, permisos]
---

# sync_files_a_nextcloud.sh

Este script monta un almacenamiento remoto de Nextcloud, sincroniza ficheros desde un origen local y ejecuta una correccion remota de permisos.

## Ejecucion programada

```cron
00 2 2-31 * 1-5 /ruta/origen/scripts/sync_files_a_nextcloud.sh
```

## Flujo

1. Escribe log diario.
2. Monta origen remoto Nextcloud sobre un punto de montaje destino local.
3. Sincroniza documentos desde origen local hacia destino Nextcloud.
4. Sincroniza imagenes desde origen local hacia destino Nextcloud.
5. Sincroniza herramientas desde origen local hacia destino Nextcloud.
6. Desmonta el punto de montaje.
7. Lanza por SSH un script remoto de permisos.

## Version saneada

```bash
#!/usr/bin/env bash
set -euo pipefail

LOG="/var/log/sync_nextcloud_$(date +%Y%m%d).log"
MOUNT_DESTINO="/mnt/destino/nextcloud"
NEXTCLOUD_ORIGEN="<HOSTNAME>:/ruta/origen/nextcloud-data/<USUARIO>/files"

mount "$NEXTCLOUD_ORIGEN" "$MOUNT_DESTINO"
trap 'umount -l "$MOUNT_DESTINO" 2>/dev/null || true' EXIT

rsync -avh /ruta/origen/documentos/ "$MOUNT_DESTINO/Documentos/" >> "$LOG" 2>&1
rsync -avh /ruta/origen/imagenes/ "$MOUNT_DESTINO/Imagenes/" >> "$LOG" 2>&1
rsync -avh /ruta/origen/tools/ "$MOUNT_DESTINO/tools/" >> "$LOG" 2>&1

ssh <USUARIO>@<HOSTNAME> 'sudo /ruta/origen/scripts/permisos_nextcloud.sh'
```

## Recomendaciones Nextcloud

- Usar `rsync -n` antes de sincronizar en real.
- Evitar modificar directamente `data/` si no se controla el reindexado.
- Ejecutar `occ files:scan --path <USUARIO>/files` si aplica.
- Mantener el cambio de permisos en un script remoto versionado y documentado.

<!--
Fuentes consolidadas

- `/tools/scripts/micron1`
- `/tools/scripts/sync_files_a_nextcloud.sh`
-->
