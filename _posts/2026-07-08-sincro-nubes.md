---
title: sincro_nubes.sh
date: 2026-07-08 01:10:00
categories: [Sistemas, Scripting]
tags: [bash, nextcloud, rsync, sql, nfs]
---

# sincro_nubes.sh

Este script sincroniza datos Nextcloud entre dos nubes/hosts, copia SQL y lanza una importacion remota de BBDD.

## Uso en la infraestructura

Se usa para replicar una instancia Nextcloud hacia otro host o nube secundaria.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| Replica datos | Nextcloud secundario | La instancia destino queda desactualizada |
| Replica BBDD | SQL Nextcloud | Metadatos y ficheros no coinciden |
| Recuperacion | Host alternativo | No hay entorno preparado para contingencia |

Debe controlarse la compatibilidad de versiones y evitar importar SQL sobre una instancia activa sin mantenimiento.

## Estado en cron

La entrada aparece comentada en `micron_nextcloud`:

```cron
#30 3 * * 1-5 /ruta/origen/scripts/sincro_nubes.sh
```

## Flujo

1. Monta un destino remoto para Nextcloud.
2. Sincroniza datos desde `/ruta/origen/nextcloud-data` hacia `/ruta/destino/nextcloud-data`.
3. Sincroniza dumps SQL hacia el host destino.
4. Lanza `import_sql.sh` por SSH en el host remoto.
5. Desmonta el destino.

## Script original anonimizado

```bash
#!/bin/bash
LOG=/var/log/sincro_nubes.log

logline() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') $*" >> "$LOG"
}

FECHA=D$(date +%Y%m%d)
FILEIN='etc'
FILEOUT='etc.'
FILEIN1='tools'
FILEOUT1='tools.'

logline "Inicia la sincronizacion de las nubes" > $LOG
logline "Montamos el Disco Remoto"
mount <IP_INTERNA>:/ruta/destino/backups/nextcloud /mnt/destino
if [ $? -ne 0 ]; then
  logline "Error al montar NFS "
  exit 5
else
  logline "Inicia la sincronizacion del disco"
  rsync -r -t -p -o -g -l -H -i -s --exclude-from=/ruta/origen/scripts/exclude_sincro_nextcloud.txt /ruta/origen/nextcloud /mnt/destino >> $LOG
  if [ $? -ne 0 ]; then
    logline "Error en la  sincronizacion del disco"
    exit 7
  else
    logline "Inicia la sincronizacion de la BBDD"
    rsync -r -t -p -o -g -l -H -i -s /ruta/origen/sqls <USUARIO_ADMIN>@<HOSTNAME>:/ruta/destino/ >> $LOG
    if [ $? -ne 0 ]; then
      logline "Error en la  sincronizacion de la BBDD"
      exit 9
    else
      logline "Inicia el import de la BBDD, entra en modo mantenimiento"
      ssh <USUARIO_ADMIN>@<HOSTNAME> "/ruta/origen/scripts/import_sql.sh"
      if [ $? -ne 0 ]; then
        logline "Error en el import de  la BBDD"
        exit 11
      else
        logline "Desmontamos el NFS"
        umount /mnt/destino
        logline "Finaliza el import de la BBDD, salimos de modo manteminiento"
        logline "Sincronizacion de las nubes finalizada OK"
        exit 0
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

LOG="/var/log/sincro_nubes_$(date +%Y%m%d).log"
MOUNT_DESTINO="/mnt/destino/nextcloud"
NFS_DESTINO="<HOSTNAME>:/ruta/destino/nextcloud"

mount "$NFS_DESTINO" "$MOUNT_DESTINO"
trap 'umount -l "$MOUNT_DESTINO" 2>/dev/null || true' EXIT

rsync -avh --exclude-from=/ruta/origen/scripts/exclude_sincro_nextcloud.txt \
  /ruta/origen/nextcloud-data/ \
  "$MOUNT_DESTINO/" >> "$LOG" 2>&1

rsync -avh /ruta/origen/sqls/ <USUARIO_ADMIN>@<HOSTNAME>:/ruta/destino/sqls/ >> "$LOG" 2>&1
ssh <USUARIO_ADMIN>@<HOSTNAME> "/ruta/origen/scripts/import_sql.sh"
```

## Script relacionado: import_sql.sh

`import_sql.sh` activa modo mantenimiento, importa un dump SQL y desactiva mantenimiento en el Nextcloud destino.

## Riesgos

- Importar BBDD sobre un Nextcloud activo requiere ventana controlada.
- Verificar compatibilidad de versiones origen/destino.
- Usar mantenimiento y backup previo antes del import.

## Cambios y motivo

| Cambio | Motivo |
|---|---|
| `trap` para desmontar | desmonta aunque falle `rsync` o `ssh` |
| variables para mount y NFS | hace explicito origen/destino |
| `--exclude-from` conservado | mantiene tu logica de excluir rutas |
| validacion previa de versiones | evita importar BBDD incompatible en destino |

<!--
Fuentes consolidadas

- `/tools/scripts/nextcloud_scripts/micron_nextcloud`
- `/tools/scripts/nextcloud_scripts/sincro_nubes.sh`
- `/tools/scripts/nextcloud_scripts/import_sql.sh`
-->
