---
title: depuro_neptuno_90.sh
date: 2026-07-08 01:00:00
categories: [Sistemas, Scripting]
tags: [bash, limpieza, backup, find, retencion]
---

# depuro_neptuno_90.sh

Este script depura copias antiguas en el destino mensual de backups de Nextcloud.

## Uso en la infraestructura

Se usa para aplicar retencion en el almacenamiento externo y evitar que los backups llenen el destino.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| Retencion | Backups SQL/filesystem | El destino puede quedarse sin espacio |
| Limpieza diferenciada | Nextcloud data | Puede conservar o borrar mas de lo esperado |
| Control mensual | NAS/almacenamiento | La ventana de backup siguiente puede fallar |

Es una tarea sensible: antes de activar `-delete`, conviene probar el mismo `find` solo con `-print`.

## Invocacion

```bash
/ruta/origen/scripts/muevo_filesystems_neptuno.sh
```

## Flujo

1. Monta el destino remoto de backups mensuales.
2. Borra SQL, `etc`, `www` y `tools` con mas de 90 dias.
3. Borra `data` con mas de 59 dias.
4. Registra errores por bloque.

## Script original anonimizado

```bash
#!/bin/bash
LOG=/var/log/depuro_neptuno_90.log

logline() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') $*" >> "$LOG"
}

sz_ETC_Neptuno='/ruta/destino/backups-mensuales/nextcloud/etc'
sz_TOOLS_Neptuno='/ruta/destino/backups-mensuales/nextcloud/tools'
sz_WWW_Neptuno='/ruta/destino/backups-mensuales/nextcloud/www'
sz_DATA_Neptuno='/ruta/destino/backups-mensuales/nextcloud/nextcloud/data'
sz_SQLS_Neptuno='/ruta/destino/backups-mensuales/nextcloud/sqls'

logline "Inicia el proceso de depuracion de copias" > $LOG
logline "Montamos la unidad de neptuno"
mount <HOSTNAME>:/ruta/destino/backups-mensuales/nextcloud /ruta/destino/backups-mensuales/nextcloud
if [ $? -ne 0 ]; then
  logline "Error montando la unidad de neptuno"
  exit 3
else
  echo "Depuro carpeta sql" > $LOG
  find $sz_SQLS_Neptuno -mtime +90 -exec rm {} \;
  if [ $? -ne 0 ]; then logline "Error al mover sql"; exit 5; else
  logline "Depuro carpeta etc"
  find $sz_ETC_Neptuno -mtime +90 -exec rm {} \;
  if [ $? -ne 0 ]; then logline "Error al mover etc"; exit 7; else
  logline "Depuro carpeta www"
  find $sz_WWW_Neptuno -mtime +90 -exec rm {} \;
  if [ $? -ne 0 ]; then logline "Error al mover www"; exit 11; else
  logline "Depuro carpeta tools"
  find $sz_TOOLS_Neptuno -mtime +90 -exec rm {} \;
  if [ $? -ne 0 ]; then logline "Error al mover tools"; exit 13; else
  logline "Depuro carpeta data"
  find $sz_DATA_Neptuno -mtime +59 -exec rm {} \;
  if [ $? -ne 0 ]; then logline "Error al mover data"; exit 15; else
  logline "Desmontamos la unidad de neptuno, PERO YA NO SE DESMONTA PARA QUE NO COCHE CON EL BCK DE LAS 4:30"
  logline "Proceso finalizado correctamente"
  exit 0
  fi; fi; fi; fi; fi
fi
sleep 50
exit 0
```

## Version revisada

```bash
#!/usr/bin/env bash
set -euo pipefail

LOG="/var/log/depuro_neptuno_$(date +%Y%m%d).log"
MOUNT_DESTINO="/mnt/destino/backups-mensuales/nextcloud"
NFS_DESTINO="<HOSTNAME>:/ruta/destino/backups-mensuales/nextcloud"

mount "$NFS_DESTINO" "$MOUNT_DESTINO"
trap 'umount -l "$MOUNT_DESTINO" 2>/dev/null || true' EXIT

find "$MOUNT_DESTINO/sqls" -type f -mtime +90 -print -delete >> "$LOG" 2>&1
find "$MOUNT_DESTINO/etc" -type f -mtime +90 -print -delete >> "$LOG" 2>&1
find "$MOUNT_DESTINO/www" -type f -mtime +90 -print -delete >> "$LOG" 2>&1
find "$MOUNT_DESTINO/tools" -type f -mtime +90 -print -delete >> "$LOG" 2>&1
find "$MOUNT_DESTINO/nextcloud/data" -type f -mtime +59 -print -delete >> "$LOG" 2>&1
```

## Riesgos y mejoras

- Usar `-type f` para evitar borrar directorios.
- Añadir `-print` para auditar eliminaciones.
- Probar primero sin `-delete`.
- Separar retenciones por tipo de dato en variables.

## Cambios y motivo

| Cambio | Motivo |
|---|---|
| `find -type f` | evita intentar borrar directorios por accidente |
| `-print -delete` | deja rastro de lo eliminado |
| `trap` de desmontaje | controla el mount incluso si falla una limpieza |
| retenciones explicitas | hace visible por que data usa 59 dias y otros 90 |

<!--
Fuentes consolidadas

- `/tools/scripts/nextcloud_scripts/depuro_neptuno_90.sh`
- `/tools/scripts/nextcloud_scripts/muevo_filesystems_neptuno.sh`
-->
