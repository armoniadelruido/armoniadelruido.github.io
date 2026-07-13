---
title: Rsync config libra y Nextcloud
date: 2026-07-09 01:30:00
categories: [Sistemas, Scripting]
tags: [bash, rsync, nextcloud, nginx, backup]
---

# Rsync config libra y Nextcloud

Estos scripts sincronizan herramientas hacia Nextcloud y recuperan configuraciones de un host remoto.

## Uso en la infraestructura

Se usa para conservar scripts y configuraciones de un host de servicios en una ubicacion accesible desde Nextcloud o desde una maquina de administracion.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| Copia operativa | `/tools` | Se pierden scripts o no llegan a la nube |
| Configuracion | nginx/Let's Encrypt | Cuesta reconstruir reverse proxies y certificados |
| Permisos | Nextcloud | Los ficheros sincronizados pueden quedar inaccesibles |

Es backup de configuracion y soporte operativo; no sustituye el backup completo de datos.

## Script original sync Nextcloud anonimizado

```bash
#!/bin/bash
LOG=/var/log/sync_files_a_nextcloud.log

echo "Inicia la sincronizacion de documentos hacia la nube" > $LOG
echo "Monta la unidad remota" >> $LOG
mount <HOSTNAME>:/ruta/destino/nextcloud/files /mnt/nextcloud >> $LOG
if [ $? -ne 0 ]
then
        echo "Error montando la unidad remota" > $LOG
        exit 5
else
        echo "Sincronizamos las rutas" >> $LOG
        rsync -r -t -p -o -g -l -H -i -s /ruta/origen/tools /mnt/nextcloud/libra/ >> $LOG
if [ $? -ne 0 ]
then
        echo "Error en la  sincronizacion" >> $LOG
        exit 7
else
        echo "Desmonta la unidad remota" >> $LOG
        umount -l /mnt/nextcloud >> $LOG
if [ $? -ne 0 ]
then
        echo "Error desmontando unidad remota" >> $LOG
        exit 9
else
        echo "Ejecutamos script remoto para permisos ficheros" >> $LOG
        su - <USUARIO> -c "ssh <USUARIO>@<HOSTNAME> 'sudo /ruta/origen/scripts/permiso_ficheros_fisher_libra.sh'"
if [ $? -ne 0 ]
then
        echo "Problemas con el cambio de permisos" >> $LOG
        exit 11
else
        echo "Sincronizacion finalizada con exito" >> $LOG
fi
fi
fi
fi
exit 0
```

## Script original recuperar configuracion anonimizado

```bash
#!/bin/bash

rsync -avzie ssh <HOSTNAME>:/ruta/origen/nginx/sites-enabled/ /ruta/destino/nginx/sites-enabled
rsync -avzie ssh <HOSTNAME>:/ruta/origen/nginx/sites-available/ /ruta/destino/nginx/sites-available
rsync -avzie ssh <HOSTNAME>:/ruta/origen/letsencrypt/ /ruta/destino/letsencrypt

exit 0
```

## Version revisada de sync hacia Nextcloud

```bash
#!/usr/bin/env bash
set -euo pipefail

LOG="/var/log/sync_files_a_nextcloud_$(date +%Y%m%d).log"
MOUNT_DESTINO="/mnt/destino/nextcloud"
NFS_DESTINO="<HOSTNAME>:/ruta/destino/nextcloud/files"

mount "$NFS_DESTINO" "$MOUNT_DESTINO"
trap 'umount -l "$MOUNT_DESTINO" 2>/dev/null || true' EXIT

rsync -rtpoglisH /ruta/origen/tools/ "$MOUNT_DESTINO/libra/tools/" >> "$LOG" 2>&1

ssh <USUARIO>@<HOSTNAME> 'sudo /ruta/origen/scripts/permiso_ficheros.sh'
```

## Version revisada de recuperar configuracion

```bash
#!/usr/bin/env bash
set -euo pipefail

rsync -avz -e ssh <HOSTNAME>:/ruta/origen/nginx/sites-enabled/ /ruta/destino/nginx/sites-enabled/
rsync -avz -e ssh <HOSTNAME>:/ruta/origen/nginx/sites-available/ /ruta/destino/nginx/sites-available/
rsync -avz -e ssh <HOSTNAME>:/ruta/origen/letsencrypt/ /ruta/destino/letsencrypt/
```

## Recomendaciones

- Usar barras finales de forma consistente en `rsync`.
- Evitar montar en rutas compartidas si hay varias tareas concurrentes.
- Probar con `rsync -n` antes de activar sincronizacion bidireccional o destructiva.
- No copiar claves de Let's Encrypt sin cifrado o permisos restrictivos.

## Cambios y motivo

| Cambio | Motivo |
|---|---|
| `trap` para desmontar | desmonta aunque falle `rsync` o el SSH remoto |
| barras finales en `rsync` | aclara si se copia carpeta o contenido |
| rutas origen/destino explicitas | separa configuracion recuperada de backup de datos |
| advertencia sobre Let's Encrypt | contiene claves privadas y requiere permisos estrictos |

<!--
Fuentes consolidadas

- `/tools/scripts/libra_scripts/sync_files_a_nextcloud.sh`
- `/tools/scripts/libra_scripts/me_traigo_la_config_de_libra.sh`
-->
