---
title: liberamemoria.sh
date: 2026-07-07 00:10:00
categories: [Sistemas, Scripting]
tags: [bash, memoria, linux, mantenimiento, cron]
---

# liberamemoria.sh

Este script comprueba memoria libre y, si baja de un umbral, fuerza la liberacion de caches del kernel Linux.

## Ejecucion programada

```cron
@hourly /ruta/origen/scripts/liberamemoria.sh
```

## Flujo

1. Obtiene memoria libre con `free -m`.
2. Compara el valor con un umbral en MB.
3. Si la memoria libre es inferior o igual al umbral, escribe `3` en `/proc/sys/vm/drop_caches`.

## Version saneada

```bash
#!/usr/bin/env bash
set -euo pipefail

UMBRAL_MB="${UMBRAL_MB:-6144}"
MEMORIA_LIBRE="$(free -m | awk '/^Mem:/ {print $4}')"

if (( MEMORIA_LIBRE <= UMBRAL_MB )); then
  sync
  echo 3 > /proc/sys/vm/drop_caches
fi
```

## Riesgos y mejoras

- Requiere permisos de root.
- `drop_caches` no soluciona fugas de memoria; solo libera caches recuperables.
- Conviene registrar ejecuciones en log para detectar frecuencia excesiva.
- La version original intenta guardar un comando en una variable de forma incorrecta; mejor ejecutar `sync` y `echo` directamente.

<!--
Fuentes consolidadas

- `/tools/scripts/micron1`
- `/tools/scripts/liberamemoria.sh`
-->
