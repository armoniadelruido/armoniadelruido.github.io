---
title: backup_nube_neptuno.sh
date: 2026-07-08 01:20:00
categories: [Sistemas, Scripting]
tags: [bash, nextcloud, rsync, backup, nfs]
---

# backup_nube_neptuno.sh

Este script sincroniza el arbol de datos de Nextcloud hacia un destino remoto mensual.

## Uso en la infraestructura

Se usa para conservar una copia mensual del arbol de datos de Nextcloud en almacenamiento externo.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| Backup datos | Nextcloud files | No hay historico mensual de ficheros |
| Copia externa | NAS/almacenamiento | El host local es punto unico de fallo |
| Capacidad | Destino mensual | Puede llenarse si no se combina con retencion |

Complementa al dump SQL: por si solo no basta para una restauracion completa.

## Estado en cron

La entrada aparece comentada en `micron_nextcloud`:

```cron
#30 4 1 * * /ruta/origen/scripts/backup_nube_neptuno.sh
```

## Flujo

1. Monta un destino remoto mensual.
2. Sincroniza `/ruta/origen/nextcloud-data` hacia `/ruta/destino/backups-mensuales/nextcloud`.
3. Entra al directorio destino.
4. Desmonta el destino.

## Script original

```bash
#!/bin/bash
LOG=/var/log/backup_nube_neptuno.log

logline() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') $*" >> "$LOG"
}

FECHA=D$(date +%Y%m%d)
FILEIN='nextcloud'
FILEOUT='nextcloud.'

logline "Inicia el proceso de empaque de nextcloud" > $LOG
logline "Montamos la unidad de red"
mount <HOSTNAME>:/ruta/destino/backups-mensuales/nextcloud /mnt/destino
if [ $? -ne 0 ]; then
  logline "Error montando la unidad de red"
  exit 3
else
  logline "Inicia la sincronizacion de paquetes" > $LOG
  rsync -r -t -p -o -g -l -H -i -s /ruta/origen/nextcloud /mnt/destino >> $LOG
  if [ $? -ne 0 ]; then
    logline "Error en rsync"
    exit 5
  else
    logline "Nos ponemos en el directorio"
    cd /mnt/destino/
    if [ $? -ne 0 ]; then
      logline "Error al mover etc"
      exit 7
    else
      logline "Desmontamos la unidad de red"
      umount -l /mnt/destino
      if [ $? -ne 0 ]; then
        logline "Error desmontando unidad"
        exit 11
      else
        logline "Proceso finalizado correctamente"
        exit 0
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

LOG="/var/log/backup_nube_neptuno_$(date +%Y%m%d).log"
ORIGEN_NEXTCLOUD="/ruta/origen/nextcloud-data"
MOUNT_DESTINO="/mnt/destino/backups-mensuales/nextcloud"
NFS_DESTINO="<HOSTNAME>:/ruta/destino/backups-mensuales/nextcloud"

mount "$NFS_DESTINO" "$MOUNT_DESTINO"
trap 'umount -l "$MOUNT_DESTINO" 2>/dev/null || true' EXIT

rsync -avh "$ORIGEN_NEXTCLOUD/" "$MOUNT_DESTINO/nextcloud/" >> "$LOG" 2>&1
```

## Recomendaciones

- Usar `--delete` solo si el destino debe ser espejo exacto.
- Usar `--link-dest` o snapshots si se quieren historicos eficientes.
- Validar espacio libre antes de copiar.

## Cambios y motivo

| Cambio | Motivo |
|---|---|
| elimina `cd` innecesario | no aporta validacion real del backup |
| `trap` para desmontar | evita mounts colgados |
| variables origen/destino | aclara que se copia data hacia backup mensual |
| log unico | evita pisar el log con redirecciones `>` intermedias |

<!--
Fuentes consolidadas

- `/tools/scripts/nextcloud_scripts/micron_nextcloud`
- `/tools/scripts/nextcloud_scripts/backup_nube_neptuno.sh`
-->
