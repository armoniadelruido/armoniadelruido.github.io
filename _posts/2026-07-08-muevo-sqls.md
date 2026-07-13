---
title: muevo_sqls.sh
date: 2026-07-08 00:40:00
categories: [Sistemas, Scripting]
tags: [bash, sql, backup, tar, nfs]
---

# muevo_sqls.sh

Este script empaqueta dumps SQL desde un origen local, monta un destino remoto y mueve los tarballs al almacenamiento mensual.

## Uso en la infraestructura

Se usa para sacar dumps SQL del host y conservarlos en almacenamiento externo con retencion separada.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| Archivado SQL | Backups Nextcloud | Los dumps quedan solo en local |
| Copia externa | NAS/almacenamiento | Se pierde proteccion ante fallo del host |
| Limpieza origen | Disco local | Puede llenarse si no se depura |

No debe borrar dumps locales hasta confirmar que el tarball llego al destino.

## Ejecucion programada

```cron
15 3 * * 0 /ruta/origen/scripts/muevo_sqls.sh
```

## Flujo

1. Entra en `/ruta/origen/backups-locales`.
2. Comprime la carpeta `sqls` en `/tmp`.
3. Monta un destino remoto de backups mensuales.
4. Mueve el tarball a `/ruta/destino/backups-mensuales/sqls`.
5. Desmonta el destino.
6. Elimina los dumps SQL del origen local.

## Script original

```bash
#!/bin/bash
LOG=/var/log/muevo_sqls.log

logline() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') $*" >> "$LOG"
}

FECHA=D$(date +%Y%m%d)
FILEIN='sqls'
FILEOUT='sqls.'
cd /ruta/origen/backups-locales/

logline "Inicia la compresion de de los filesystems" > $LOG
tar cvpzPf /tmp/$FILEOUT$FECHA.tar.gz $FILEIN >> $LOG
if [ $? -ne 0 ]; then
  logline "Error en la  sincronizacion de tools"
  exit 5
else
  logline "Arhivos tartarizados"
  logline "Montamos la unidad de red"
  mount <HOSTNAME>:/ruta/destino/backups-mensuales/nextcloud /mnt/destino
  if [ $? -ne 0 ]; then
    logline "Error montando la unidad de red"
    exit 7
  else
    logline "Unidad Montada"
    logline "Muevo el tar a la unidad de red"
    mv /tmp/sqls*tar.gz /mnt/destino/sqls
    if [ $? -ne 0 ]; then
      logline "Error moviendo archivos"
      exit 9
    else
      logline "Desmontamos la unidad de red"
      umount -l /mnt/destino
      if [ $? -ne 0 ]; then
        logline "Error desmontando unidad"
        exit 11
      else
        logline "Eliminamos archivos en origen"
        rm -f /ruta/origen/backups-locales/sqls/bck_*
        if [ $? -ne 0 ]; then
          logline "Error eliminando archivos"
          exit 13
        else
          logline "Proceso finalizado correctamente"
          exit 0
        fi
      fi
    fi
  fi
fi
exit 0
```

## Version revisada

```bash
#!/usr/bin/env bash
set -euo pipefail

LOG="/var/log/muevo_sqls_$(date +%Y%m%d).log"
FECHA="$(date +%Y%m%d)"
ORIGEN_SQL="/ruta/origen/backups-locales/sqls"
TMP_TAR="/tmp/sqls_${FECHA}.tar.gz"
MOUNT_DESTINO="/mnt/destino/backups-mensuales"
NFS_DESTINO="<HOSTNAME>:/ruta/destino/backups-mensuales"

tar czf "$TMP_TAR" -C "$(dirname "$ORIGEN_SQL")" "$(basename "$ORIGEN_SQL")"
mount "$NFS_DESTINO" "$MOUNT_DESTINO"
trap 'umount -l "$MOUNT_DESTINO" 2>/dev/null || true' EXIT
mv "$TMP_TAR" "$MOUNT_DESTINO/sqls/"
find "$ORIGEN_SQL" -type f -name 'bck_*.sql*' -print -delete >> "$LOG" 2>&1
```

## Riesgos y mejoras

- No borrar origen hasta confirmar copia en destino.
- Usar `find -type f` y patrones concretos.
- Registrar el tamaño del tar final.
- Evitar `mv /tmp/sqls*tar.gz` por ser demasiado amplio.

## Cambios y motivo

| Cambio | Motivo |
|---|---|
| `trap` para desmontar | evita dejar el NFS montado si falla un paso intermedio |
| tarball con nombre exacto | evita mover ficheros de `/tmp` que no pertenecen al backup |
| `find -type f -name` | limita el borrado a ficheros esperados |
| no borrar antes de validar destino | reduce riesgo de perder dumps validos |

<!--
Fuentes consolidadas

- `/tools/scripts/nextcloud_scripts/micron_nextcloud`
- `/tools/scripts/nextcloud_scripts/muevo_sqls.sh`
-->
