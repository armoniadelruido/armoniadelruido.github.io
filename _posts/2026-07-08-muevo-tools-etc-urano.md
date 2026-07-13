---
title: muevo_tools_etc_urano.sh
date: 2026-07-08 01:30:00
categories: [Sistemas, Scripting]
tags: [bash, backup, tar, tools, etc]
---

# muevo_tools_etc_urano.sh

Este script comprime los directorios `etc` y `tools` desde un origen local de backups y mueve los tarballs hacia un destino externo.

## Uso en la infraestructura

Se usa para guardar configuracion del sistema y scripts operativos fuera del host principal.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| Config sistema | `/etc` | Rehacer el host requiere mas trabajo manual |
| Automatizacion | `/tools` | Se pierden scripts de operacion y mantenimiento |
| Copia externa | Almacenamiento remoto | No hay copia mensual fuera del host |

Es parte del backup de configuracion, no del backup de datos de usuario.

## Estado en cron

La entrada aparece comentada en `micron_nextcloud`:

```cron
#15 3 * * 0 /ruta/origen/scripts/muevo_tools_etc_urano.sh
```

## Flujo

1. Entra en `/ruta/origen/backups`.
2. Comprime `etc`.
3. Comprime `tools`.
4. Mueve ambos tarballs a `/ruta/destino/backups`.

## Script original

```bash
#!/bin/bash
LOG=/var/log/muevo_tools_etc_urano.log

logline() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') $*" >> "$LOG"
}

FECHA=D$(date +%Y%m%d)
FILEIN='etc'
FILEOUT='etc.'
FILEIN1='tools'
FILEOUT1='tools.'
cd /ruta/origen/backups

logline "Inicia el proceso de empaque los filesystems" > $LOG
logline "Inicia la compresion de etc"
tar cvpzf $FILEOUT$FECHA.tar.gz $FILEIN >> $LOG
if [ $? -ne 0 ]; then logline "Error al comprimir etc"; exit 3; else
logline "Inicia la compresion de tools"
tar cvpzf $FILEOUT1$FECHA.tar.gz $FILEIN1 >> $LOG
if [ $? -ne 0 ]; then logline "Error al comprimir tools"; exit 5; else
logline "Mueve etc a Urano"
mv etc*tar.gz /ruta/destino/backups/
if [ $? -ne 0 ]; then logline "Error al mover etc"; exit 7; else
logline "Mueve tools a Urano"
mv tools*tar.gz /ruta/destino/backups/
if [ $? -ne 0 ]; then logline "Error al mover tools"; exit 9; else
logline "Proceso finalizado con exito"
fi; fi; fi; fi
exit 0
```

## Version revisada

```bash
#!/usr/bin/env bash
set -euo pipefail

FECHA="$(date +%Y%m%d)"
ORIGEN_BASE="/ruta/origen/backups"
DESTINO_BASE="/ruta/destino/backups"

tar czf "$ORIGEN_BASE/etc.${FECHA}.tar.gz" -C "$ORIGEN_BASE" etc
tar czf "$ORIGEN_BASE/tools.${FECHA}.tar.gz" -C "$ORIGEN_BASE" tools

mv "$ORIGEN_BASE/etc.${FECHA}.tar.gz" "$DESTINO_BASE/etc/"
mv "$ORIGEN_BASE/tools.${FECHA}.tar.gz" "$DESTINO_BASE/tools/"
```

## Riesgos y mejoras

- Validar que el destino esta montado antes de mover.
- Evitar comodines amplios como `etc*tar.gz` si hay mas ficheros similares.
- Registrar tamano y checksum de cada tarball.

## Cambios y motivo

| Cambio | Motivo |
|---|---|
| nombres exactos para tarballs | evita mover ficheros antiguos por comodin |
| variables origen/destino | facilita cambiar ubicaciones |
| `set -euo pipefail` | simplifica los `if [ $? -ne 0 ]` encadenados |
| destino validado | evita mover a una ruta no montada |

<!--
Fuentes consolidadas

- `/tools/scripts/nextcloud_scripts/micron_nextcloud`
- `/tools/scripts/nextcloud_scripts/muevo_tools_etc_urano.sh`
-->
