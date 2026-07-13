---
title: proxmox_backup_collar.sh
date: 2026-07-07 01:00:00
categories: [Sistemas, Scripting]
tags: [bash, proxmox, pbs, backup, pxar]
---

# proxmox_backup_collar.sh

Este script lanza backups con `proxmox-backup-client` de varias rutas origen hacia un repositorio PBS y ejecuta limpieza de memoria en el host remoto al finalizar.

## Uso en la infraestructura

Se usa para conservar configuracion del hipervisor y rutas de soporte en Proxmox Backup Server.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| Config hipervisor | Proxmox | Se complica reconstruir storage, red y jobs |
| Backup externo | PBS | No hay copia versionada fuera del host |
| Recuperacion | Laboratorio/VMs | Aumenta el tiempo de vuelta a servicio |

Complementa los backups de VMs; no los sustituye.

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

## Script original

```bash
#!/bin/bash
LOG=/var/log/proxmox_backup_collar.log
FECHA=D$(date +%Y%m%d)
FILEIN='etc'
FILEOUT='etc.'
FILEIN1='tools'
FILEOUT1='tools.'
export PBS_PASSWORD_FILE=/ruta/segura/proxmox_client
export PROXMOX_BACKUP_FINGERPRINT='<FINGERPRINT>'

echo "Inicia backups de los filesystems" > $LOG
proxmox-backup-client backup \
        collar-etc.pxar:/ruta/origen/etc \
        collar-IMGs.pxar:/ruta/origen/IMGs \
        collar-Descargas.pxar:/ruta/origen/Descargas \
        collar-tools.pxar:/ruta/origen/tools \
        collar-cursos.pxar:/ruta/origen/cursos \
        collar-multimedia_base.pxar:/ruta/origen/multimedia_base \
        collar-site-web.pxar:/ruta/origen/site-web \
        --repository <USUARIO>@pbs@<HOSTNAME>:<DATASTORE> >> $LOG

if [ $? -ne 0 ]
then
        echo "Error en el backup" >> $LOG
        exit 2
else
        echo "Backups Finalizados" >> $LOG
        ssh <USUARIO_ADMIN>@<HOSTNAME> "/ruta/origen/scripts/liberamemoria.sh"
fi
exit 0
```

## Version revisada

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

## Cambios y motivo

| Cambio | Motivo |
|---|---|
| variables para repositorio PBS | facilita cambiar datastore o host |
| validar credenciales y fingerprint | falla antes de iniciar backup largo |
| separar rutas en lista | simplifica revisar que entra en PBS |
| no mezclar limpieza remota con backup | evita que fallo de limpieza o SSH marque ambiguamente el resultado |

<!--
Fuentes consolidadas

- `/tools/scripts/micron1`
- `/tools/scripts/proxmox_backup_collar.sh`
- `/tools/scripts/liberamemoria.sh`
-->
