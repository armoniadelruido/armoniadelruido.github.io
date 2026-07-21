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

Debe intentar enviar el tarball al destino externo, pero la retencion local no debe depender de que el mount remoto funcione.

## Ejecucion programada

```cron
15 3 * * 0 /ruta/origen/scripts/muevo_sqls.sh
```

## Flujo

1. Entra en `/ruta/origen/backups-locales`.
2. Comprime la carpeta `sqls` en `/tmp`.
3. Monta un destino remoto de backups mensuales con timeout.
4. Si el destino monta, mueve el tarball a `/ruta/destino/backups-mensuales/sqls`.
5. Desmonta el destino.
6. Siempre al salir, aplica retencion local: ultimos 15 dumps y ultimos 2 tar SQL en `/tmp`.

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
RETENER_DUMPS=15
RETENER_TARS=2

logline() {
  printf '%s %s\n' "$(date '+%Y-%m-%d %H:%M:%S')" "$*" >> "$LOG"
}

limpia_sqls_locales() {
  find "$ORIGEN_SQL" -maxdepth 1 -type f -name 'bck_*' -printf '%T@ %p\n' \
    | sort -rn \
    | awk -v keep="$RETENER_DUMPS" 'NR>keep { $1=""; sub(/^ /, ""); print }' \
    | while IFS= read -r fichero; do
        [[ -n "$fichero" ]] && rm -f -- "$fichero"
      done
}

limpia_sqls_tmp() {
  find /tmp -maxdepth 1 -type f -name 'sqls.*.tar.gz' -printf '%T@ %p\n' \
    | sort -rn \
    | awk -v keep="$RETENER_TARS" 'NR>keep { $1=""; sub(/^ /, ""); print }' \
    | while IFS= read -r fichero; do
        [[ -n "$fichero" ]] && rm -f -- "$fichero"
      done
}

limpieza_salida() {
  limpia_sqls_locales
  limpia_sqls_tmp
  umount -l "$MOUNT_DESTINO" 2>/dev/null || true
}

trap limpieza_salida EXIT

tar czf "$TMP_TAR" -C "$(dirname "$ORIGEN_SQL")" "$(basename "$ORIGEN_SQL")"
timeout 60s mount "$NFS_DESTINO" "$MOUNT_DESTINO"
mv "$TMP_TAR" "$MOUNT_DESTINO/sqls/"
```

## Riesgos y mejoras

- La retencion local debe ejecutarse aunque falle el destino externo.
- Usar `find -type f` y patrones concretos.
- Registrar el tamaño del tar final.
- Evitar `mv /tmp/sqls*tar.gz` por ser demasiado amplio.
- El mount NFS debe tener timeout para no bloquear el cron.
- `/tmp` tambien necesita retencion si el destino no monta.

## Cambios y motivo

| Cambio | Motivo |
|---|---|
| `trap` para desmontar | evita dejar el NFS montado si falla un paso intermedio |
| tarball con nombre exacto | evita mover ficheros de `/tmp` que no pertenecen al backup |
| `find -type f -name` | limita el borrado a ficheros esperados |
| `timeout 60s mount` | evita que NFS bloquee el cron durante demasiado tiempo |
| retener 15 dumps locales | evita llenar el origen local conservando margen de recuperacion |
| retener 2 tar en `/tmp` | evita llenar temporales si falla el envio externo |
| `trap` de limpieza | aplica retencion tanto en exito como en fallo |

<!--
Fuentes consolidadas

- `/tools/scripts/nextcloud_scripts/micron_nextcloud`
- `/tools/scripts/nextcloud_scripts/muevo_sqls.sh`
-->
