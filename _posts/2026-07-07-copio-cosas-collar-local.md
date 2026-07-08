---
title: copio_cosas_collar_local.sh
date: 2026-07-07 00:40:00
categories: [Sistemas, Scripting]
tags: [bash, rsync, backup, homelab, logs]
---

# copio_cosas_collar_local.sh

Este script sincroniza varias rutas origen locales hacia un destino local de backup usando `rsync` y log diario.

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

## Version parametrizada

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

<!--
Fuentes consolidadas

- `/tools/scripts/micron1`
- `/tools/scripts/copio_cosas_collar_local.sh`
-->
