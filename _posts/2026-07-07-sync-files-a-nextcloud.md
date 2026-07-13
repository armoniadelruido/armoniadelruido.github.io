---
title: sync_files_a_nextcloud.sh
date: 2026-07-07 00:50:00
categories: [Sistemas, Scripting]
tags: [bash, rsync, nextcloud, nfs, permisos]
---

# sync_files_a_nextcloud.sh

Este script monta un almacenamiento remoto de Nextcloud, sincroniza ficheros desde un origen local y ejecuta una correccion remota de permisos.

## Uso en la infraestructura

Se usa para publicar o respaldar ficheros locales en Nextcloud manteniendo permisos corregidos en destino.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| Sincronizacion | Nextcloud files | Los ficheros locales no llegan a la nube |
| Permisos | Nextcloud data | Los ficheros pueden no ser visibles o accesibles |
| Operacion remota | Host Nextcloud | Falla la correccion tras el rsync |

Debe coordinarse con escaneos `occ` si se modifican ficheros directamente en `data/`.

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

## Script original

```bash
#!/bin/bash
LOG=/var/log/sync_files_a_nextcloud.log

echo "Inicia la sincronizacion de documentos hacia la nube" > $LOG
echo "Monta la unidad remota" >> $LOG
mount <HOSTNAME>:/ruta/destino/nextcloud/files /mnt/nextcloud >> $LOG
if [ $? -ne 0 ]; then
        echo "Error montando la unidad remota" > $LOG
        exit 5
else
        echo "Sincronizamos Documentos" >> $LOG
        rsync -r -t -p -o -g -l -H -i -s /ruta/origen/Documentos/ /mnt/nextcloud/Documentos/ >> $LOG
if [ $? -ne 0 ]; then echo "Error en la  sincronizacion" >> $LOG; exit 7; else
        echo "Sincronizamos Imagenes" >> $LOG
        rsync -r -t -p -o -g -l -H -i -s /ruta/origen/Imagenes/ /mnt/nextcloud/Imagenes/ >> $LOG
if [ $? -ne 0 ]; then echo "Error en la  sincronizacion" >> $LOG; exit 9; else
        echo "Sincronizamos tools" >> $LOG
        rsync -r -t -p -o -g -l -H -i -s /ruta/origen/tools/ /mnt/nextcloud/halley/ >> $LOG
if [ $? -ne 0 ]; then echo "Error en la  sincronizacion" >> $LOG; exit 11; else
        echo "Desmonta la unidad remota" >> $LOG
        umount -l /mnt/nextcloud >> $LOG
if [ $? -ne 0 ]; then echo "Error desmontando unidad remota" >> $LOG; exit 13; else
        echo "Ejecutamos script remoto para permisos ficheros" >> $LOG
        su - <USUARIO> -c "ssh <USUARIO>@<HOSTNAME> 'sudo /ruta/origen/scripts/permiso_ficheros_fisher_halley.sh'"
if [ $? -ne 0 ]; then echo "Problemas con el cambio de permisos" >> $LOG; exit 15; else
        echo "Sincronizacion finalizada con exito" >> $LOG
fi; fi; fi; fi; fi; fi
exit 0
```

## Version revisada

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

## Cambios y motivo

| Cambio | Motivo |
|---|---|
| `trap` para desmontar | desmonta aunque falle un `rsync` |
| rutas en variables | hace claro que se copia a un mount Nextcloud |
| `rsync -n` recomendado | evita cambios inesperados en la nube |
| revisar `occ files:scan` | Nextcloud puede no detectar cambios directos en `data/` |

## Variante libra_scripts

La variante `sync_files_a_nextcloud.sh` monta un destino Nextcloud, sincroniza `/ruta/origen/tools` hacia una carpeta remota y ejecuta un script remoto de permisos.

<!--
Fuentes consolidadas

- `/tools/scripts/micron1`
- `/tools/scripts/sync_files_a_nextcloud.sh`
- `/tools/scripts/libra_scripts/sync_files_a_nextcloud.sh`
-->
