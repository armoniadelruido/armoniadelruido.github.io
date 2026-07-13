---
title: muevo_filesystems_neptuno.sh
date: 2026-07-08 00:50:00
categories: [Sistemas, Scripting]
tags: [bash, backup, tar, nfs, nextcloud]
---

# muevo_filesystems_neptuno.sh

Este script empaqueta varios filesystems de backup local y los mueve a un destino remoto mensual.

## Uso en la infraestructura

Se usa para consolidar copias mensuales de configuracion, aplicaciones y datos de soporte hacia almacenamiento externo.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| Archivado mensual | Filesystems de backup | Solo quedan copias locales recientes |
| Salida externa | NAS/almacenamiento | Menos resiliencia ante fallo de disco local |
| Disparo retencion | Depuracion mensual | Copias antiguas pueden acumularse |

Debe ejecutarse fuera de la ventana de dumps SQL para evitar mover copias a medio generar.

## Ejecucion programada

```cron
30 2 1 * * /ruta/origen/scripts/muevo_filesystems_neptuno.sh
```

## Flujo

1. Entra en `/ruta/origen/backups-locales`.
2. Comprime `etc`, `tools` y `www`.
3. Monta el destino remoto mensual.
4. Mueve cada tarball a su carpeta destino.
5. Llama a `depuro_neptuno_90.sh` para depurar copias antiguas.

## Script original anonimizado

```bash
#!/bin/bash
LOG=/var/log/muevo_filesystems_neptuno.log

logline() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') $*" >> "$LOG"
}

FECHA=D$(date +%Y%m%d)
FILEIN='etc'
FILEOUT='etc.'
FILEIN1='tools'
FILEOUT1='tools.'
FILEIN2='www'
FILEOUT2='www.'
cd /ruta/origen/backups-locales

logline "Inicia el proceso de empaque los filesystems" > $LOG
logline "Inicia la compresion de etc"
tar cvpzf $FILEOUT$FECHA.tar.gz $FILEIN >> $LOG
if [ $? -ne 0 ]; then logline "Error al comprimir etc"; exit 3; else
logline "Inicia la compresion de tools"
tar cvpzf $FILEOUT1$FECHA.tar.gz $FILEIN1 >> $LOG
if [ $? -ne 0 ]; then logline "Error al comprimir tools"; exit 5; else
logline "Inicia la compresion de www"
tar cvpzf $FILEOUT2$FECHA.tar.gz $FILEIN2 >> $LOG
if [ $? -ne 0 ]; then logline "Error al comprimir www"; exit 7; else
logline "Montamos la unidad de red"
mount <HOSTNAME>:/ruta/destino/backups-mensuales/nextcloud /mnt/destino
if [ $? -ne 0 ]; then logline "Error montando la unidad de red"; exit 9; else
logline "Mueve etc a Neptuno"
mv etc*tar.gz /mnt/destino/etc
if [ $? -ne 0 ]; then logline "Error al mover etc"; exit 11; else
logline "Mueve tools a Neptuno"
mv tools*tar.gz /mnt/destino/tools
if [ $? -ne 0 ]; then logline "Error al mover tools"; exit 13; else
logline "Mueve var a Neptuno"
mv www*tar.gz /mnt/destino/www
if [ $? -ne 0 ]; then logline "Error al mover var"; exit 15; else
logline "Desmontamos la unidad de red"
umount /mnt/destino
if [ $? -ne 0 ]; then logline "Error desmontando unidad"; exit 17; else
/ruta/origen/scripts/depuro_neptuno_90.sh
logline "Proceso finalizado correctamente"
exit 0
fi; fi; fi; fi; fi; fi; fi; fi
exit 0
```

## Version revisada

```bash
#!/usr/bin/env bash
set -euo pipefail

FECHA="$(date +%Y%m%d)"
ORIGEN_BASE="/ruta/origen/backups-locales"
MOUNT_DESTINO="/mnt/destino/backups-mensuales/nextcloud"
NFS_DESTINO="<HOSTNAME>:/ruta/destino/backups-mensuales/nextcloud"

for item in etc tools www; do
  tar czf "$ORIGEN_BASE/${item}.${FECHA}.tar.gz" -C "$ORIGEN_BASE" "$item"
done

mount "$NFS_DESTINO" "$MOUNT_DESTINO"
trap 'umount -l "$MOUNT_DESTINO" 2>/dev/null || true' EXIT

mv "$ORIGEN_BASE"/etc.*.tar.gz "$MOUNT_DESTINO/etc/"
mv "$ORIGEN_BASE"/tools.*.tar.gz "$MOUNT_DESTINO/tools/"
mv "$ORIGEN_BASE"/www.*.tar.gz "$MOUNT_DESTINO/www/"

/ruta/origen/scripts/depuro_neptuno_90.sh
```

## Riesgos y mejoras

- Validar que el mount destino existe antes de mover.
- Evitar comodines demasiado amplios.
- La llamada a depuracion depende de que el path del script sea correcto.
- Si el destino esta montado por otra tarea, coordinar ventanas para no desmontarlo indebidamente.

## Cambios y motivo

| Cambio | Motivo |
|---|---|
| bucle para `etc tools www` | evita repetir tres bloques casi iguales |
| rutas en variables | reduce errores al cambiar destino |
| `trap` de desmontaje | evita dejar el destino montado si falla un `mv` |
| nombres de tarballs exactos | evita comodines amplios como `etc*tar.gz` |

<!--
Fuentes consolidadas

- `/tools/scripts/nextcloud_scripts/micron_nextcloud`
- `/tools/scripts/nextcloud_scripts/muevo_filesystems_neptuno.sh`
- `/tools/scripts/nextcloud_scripts/depuro_neptuno_90.sh`
-->
