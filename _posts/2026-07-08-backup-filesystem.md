---
title: backup_filesystem.sh
date: 2026-07-08 00:20:00
categories: [Sistemas, Scripting]
tags: [bash, rsync, backup, nextcloud, filesystem]
---

# backup_filesystem.sh

Este script sincroniza rutas origen del sistema hacia un destino local de backup y replica datos de Nextcloud hacia un montaje remoto.

## Uso en la infraestructura

Se usa para proteger filesystem y datos del host Nextcloud antes de mover copias a almacenamiento externo.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| Backup local | `/tools`, `/etc`, home | Se pierden configuraciones y scripts recientes |
| Backup datos | Nextcloud data | No hay copia de ficheros de usuario |
| Copia remota | NFS/almacenamiento | Se pierde redundancia fuera del host |

Depende de que el destino remoto monte correctamente y de que `rsync` conserve permisos.

## Ejecucion programada

```cron
30 1 * * 1-5 /ruta/origen/scripts/backup_filesystem.sh
```

## Flujo

1. Crea log diario con timestamp.
2. Copia `/ruta/origen/tools` hacia `/ruta/destino/backup/tools`.
3. Copia `/ruta/origen/etc` hacia `/ruta/destino/backup/etc`.
4. Omite el backup de web si esta deshabilitado.
5. Copia `/ruta/origen/home/<USUARIO>` hacia `/ruta/destino/backup/home/<USUARIO>`.
6. Monta un destino remoto para backup de Nextcloud.
7. Sincroniza `/ruta/origen/nextcloud-data` hacia `/mnt/destino/nextcloud-backup`.
8. Desmonta el destino remoto.

## Script original anonimizado

```bash
#!/bin/bash
LOG=/var/log/backup_filesystem.log

logline() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') $*" >> "$LOG"
}

FECHA=D$(date +%Y%m%d)
FILEIN='etc'
FILEOUT='etc.'
FILEIN1='tools'
FILEOUT1='tools.'

logline "Inicia la sincronizacion de de los filesystems" > $LOG
logline "Inicia la sincronizacion tools"
rsync -r -t -p -o -g -l -H -i -s /ruta/origen/tools /ruta/destino/backups >> $LOG
if [ $? -ne 0 ]; then
  logline "Error en la  sincronizacion de tools"
  exit 5
else
  logline "Inicia la sincronizacion de etc"
  rsync -r -t -p -o -g -l -H -i -s /ruta/origen/etc /ruta/destino/backups >> $LOG
  if [ $? -ne 0 ]; then
    logline "Error en la  sincronizacion de etc"
    exit 7
  else
    logline "Inicia la sincronizacion de var"
    logline "Ya no hago el backup de www"
    if [ $? -ne 0 ]; then
      logline "Error en la  sincronizacion de var"
      exit 9
    else
      logline "Inicia la sincronizacion del home de <USUARIO>"
      rsync -r -t -p -o -g -l -H -i -s /ruta/origen/home/<USUARIO> /ruta/destino/backups >> $LOG
      if [ $? -ne 0 ]; then
        logline "Error en la  sincronizacion del home de <USUARIO>"
        exit 11
      else
        logline "Inicia la sincronizacion de nextcloud"
        mount <HOSTNAME>:/ruta/destino/nextcloud-backup /mnt/destino
        rsync -r -t -p -o -g -l -H -i -s /ruta/origen/nextcloud-data/ /mnt/destino >> $LOG
        sleep 200
        umount -l /mnt/destino
        if [ $? -ne 0 ]; then
          logline "Error en la  sincronizacion de nextcloud"
          exit 13
        else
          logline "Finaliza la sincronizacion"
        fi
      fi
    fi
  fi
fi
sleep 50
exit 0
```

## Version revisada

```bash
#!/usr/bin/env bash
set -euo pipefail

LOG="/var/log/backup_filesystem_$(date +%Y%m%d).log"
DESTINO_LOCAL="/ruta/destino/backup"
MOUNT_DESTINO="/mnt/destino/nextcloud-backup"
NFS_DESTINO="<HOSTNAME>:/ruta/destino/nextcloud-backup"

logline() {
  printf '[%s] %s\n' "$(date '+%Y-%m-%d %H:%M:%S')" "$*" >> "$LOG"
}

rsync -avh /ruta/origen/tools/ "$DESTINO_LOCAL/tools/" >> "$LOG" 2>&1
rsync -avh /ruta/origen/etc/ "$DESTINO_LOCAL/etc/" >> "$LOG" 2>&1
rsync -avh /ruta/origen/home/<USUARIO>/ "$DESTINO_LOCAL/home/<USUARIO>/" >> "$LOG" 2>&1

mount "$NFS_DESTINO" "$MOUNT_DESTINO"
trap 'umount -l "$MOUNT_DESTINO" 2>/dev/null || true' EXIT
rsync -avh /ruta/origen/nextcloud-data/ "$MOUNT_DESTINO/" >> "$LOG" 2>&1
```

## Mejoras recomendadas

- Usar `trap` para desmontar aunque falle `rsync`.
- Evitar sleeps fijos y comprobar estado del mount.
- Parametrizar origenes y destinos.
- Usar `rsync -n` antes de activar cambios destructivos.

## Cambios y motivo

| Cambio | Motivo |
|---|---|
| `trap` en el mount | desmonta aunque falle `rsync` |
| variables para destinos | evita rutas repetidas y facilita migraciones |
| elimina `sleep` como control | el estado del comando es mas fiable que esperar |
| `rsync -avh` | simplifica flags manteniendo permisos y recursividad |

<!--
Fuentes consolidadas

- `/tools/scripts/nextcloud_scripts/micron_nextcloud`
- `/tools/scripts/nextcloud_scripts/backup_filesystem.sh`
-->
