---
title: copio_cosas_collar_local.sh
date: 2026-07-07 00:40:00
categories: [Sistemas, Scripting]
tags: [bash, rsync, backup, homelab, logs]
---

# copio_cosas_collar_local.sh

Este script sincroniza varias rutas origen locales hacia un destino local de backup usando `rsync` y log diario.

## Uso en la infraestructura

Se usa para copias locales rapidas de rutas personales o de sistema antes de enviarlas a otro destino.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| Backup local | Documentos/configs | Se pierde copia rapida en el mismo entorno |
| Pre-stage | Backups externos | Las tareas posteriores no tienen origen actualizado |
| Auditoria | Logs de copia | No queda trazabilidad de errores rsync |

Es una capa local; no protege por si sola ante perdida completa del host.

## Ejecucion programada

```cron
30 1 * * 1-5 /ruta/origen/scripts/copio_cosas_collar_local.sh
```

## Flujo

1. Inicializa log diario.
2. Sincroniza imagenes desde ruta origen a destino multimedia.
3. Sincroniza musica desde ruta origen a destino multimedia.
4. Sincroniza videos desde ruta origen a destino multimedia.
5. Sincroniza documentos desde ruta origen a destino multimedia.
6. Sincroniza descargas, herramientas, configuracion y site web hacia destino de backup.
7. Sale con codigos diferentes segun el bloque que falle.

## Script original

```bash
#!/bin/bash
LOG=/var/log/copio_cosas_collar_local.log

echo "Inicia la sincronizacion del collar" > $LOG
echo "Inicia la sincronizacion de Imagenes" >> $LOG
rsync -r -t -p -o -g -l -H -i -s /ruta/origen/Imagenes /ruta/destino/multimedia_base/ >> $LOG
if [ $? -ne 0 ]; then echo "Error en la  sincronizacion de Imagenes" >> $LOG; exit 5; else
echo "Inicia la sincronizacion de Musica" >> $LOG
rsync -r -t -p -o -g -l -H -i -s /ruta/origen/Musica /ruta/destino/multimedia_base/ >> $LOG
if [ $? -ne 0 ]; then echo "Error en la  sincronizacion de Musica" >> $LOG; exit 5; else
echo "Inicia la sincronizacion de Videos" >> $LOG
rsync -r -t -p -o -g -l -H -i -s /ruta/origen/Videos /ruta/destino/multimedia_base/ >> $LOG
if [ $? -ne 0 ]; then echo "Error en la  sincronizacion de Videos" >> $LOG; exit 7; else
echo "Inicia la sincronizacion de Documentos" >> $LOG
rsync -r -t -p -o -g -l -H -i -s /ruta/origen/Documentos /ruta/destino/multimedia_base/ >> $LOG
if [ $? -ne 0 ]; then echo "Error en la  sincronizacion de Documentos" >> $LOG; exit 9; else
echo "Inicia la sincronizacion de Descargas" >> $LOG
rsync -r -t -p -o -g -l -H -i -s /ruta/origen/Descargas /ruta/destino/ >> $LOG
if [ $? -ne 0 ]; then echo "Error en la  sincronizacion de Descargas" >> $LOG; exit 11; else
echo "Inicia la sincronizacion de tools" >> $LOG
rsync -r -t -p -o -g -l -H -i -s /ruta/origen/tools /ruta/destino/ >> $LOG
if [ $? -ne 0 ]; then echo "Error en la  sincronizacion de tools" >> $LOG; exit 13; else
echo "Inicia la sincronizacion de etc" >> $LOG
rsync -r -t -p -o -g -l -H -i -s /ruta/origen/etc /ruta/destino/ >> $LOG
if [ $? -ne 0 ]; then echo "Error en la  sincronizacion de etc" >> $LOG; exit 15; else
echo "Inicia la sincronizacion de site-web" >> $LOG
rsync -r -t -p -o -g -l -H -i -s /ruta/origen/site-web /ruta/destino/ >> $LOG
if [ $? -ne 0 ]; then echo "Error en la  sincronizacion de site-web" >> $LOG; exit 17; else
echo "Finaliza la sincronizacion" >> $LOG
fi; fi; fi; fi; fi; fi; fi; fi
exit 0
```

## Version revisada parametrizada

```bash
#!/usr/bin/env bash
set -euo pipefail

LOG="/var/log/backup_local_$(date +%Y%m%d).log"
DESTINO_BASE="/ruta/destino/backup"

declare -A sincronizaciones=(
  ["/ruta/origen/imagenes"]="$DESTINO_BASE/multimedia/imagenes"
  ["/ruta/origen/musica"]="$DESTINO_BASE/multimedia/musica"
  ["/ruta/origen/videos"]="$DESTINO_BASE/multimedia/videos"
  ["/ruta/origen/documentos"]="$DESTINO_BASE/multimedia/documentos"
  ["/ruta/origen/descargas"]="$DESTINO_BASE/descargas"
  ["/ruta/origen/tools"]="$DESTINO_BASE/tools"
  ["/ruta/origen/etc"]="$DESTINO_BASE/etc"
  ["/ruta/origen/site-web"]="$DESTINO_BASE/site-web"
)

for origen in "${!sincronizaciones[@]}"; do
  destino="${sincronizaciones[$origen]}"
  printf '[%s] %s -> %s\n' "$(date --iso-8601=seconds)" "$origen" "$destino" >> "$LOG"
  rsync -avh --delete "$origen/" "$destino/" >> "$LOG" 2>&1
done
```

## Riesgos y mejoras

- Usar arrays reduce anidamiento de `if`.
- `--dry-run` es recomendable para probar cambios.
- Las rutas origen/destino deben estar entrecomilladas.
- Si se usa `--delete`, confirmar que el destino es correcto.

## Cambios y motivo

| Cambio | Motivo |
|---|---|
| lista de rutas | evita repetir bloques `rsync` casi identicos |
| `for` sobre origenes | simplifica añadir o quitar carpetas |
| comillas en rutas | soporta nombres con espacios o caracteres especiales |
| log consistente | reduce redirecciones repetidas y facilita revisar errores |

<!--
Fuentes consolidadas

- `/tools/scripts/micron1`
- `/tools/scripts/copio_cosas_collar_local.sh`
-->
