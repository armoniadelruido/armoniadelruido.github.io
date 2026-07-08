---
title: depura_ficheros.sh
date: 2026-07-07 00:30:00
categories: [Sistemas, Scripting]
tags: [bash, limpieza, backups, find, pfsense]
---

# depura_ficheros.sh

Este script elimina backups antiguos de varias carpetas destino de pfSense.

## Invocacion

Se llama desde:

```bash
/ruta/origen/scripts/pfsense/pfsense_backups_auto.sh
```

## Flujo

1. Define tres rutas destino de backups.
2. Ejecuta `find` en cada destino.
3. Elimina ficheros con mas de 15 dias.
4. Registra errores con codigos de salida diferentes por carpeta.

## Version saneada

```bash
#!/usr/bin/env bash
set -euo pipefail

LOG="/var/log/depura_pfsense_$(date +%Y%m%d).log"
DIAS_RETENCION="${DIAS_RETENCION:-15}"

destinos=(
  "/ruta/destino/backups/pfsense/fw1"
  "/ruta/destino/backups/pfsense/fw2"
  "/ruta/destino/backups/pfsense/fw3"
)

for destino in "${destinos[@]}"; do
  printf '[%s] Depurando %s\n' "$(date --iso-8601=seconds)" "$destino" >> "$LOG"
  find "$destino" -type f -name '*.xml' -mtime +"$DIAS_RETENCION" -print -delete >> "$LOG" 2>&1
done
```

## Riesgos y mejoras

- Añadir `-type f` para evitar borrar directorios.
- Registrar con `-print` lo eliminado.
- Entrecomillar variables.
- Permitir `DIAS_RETENCION` configurable.
- Probar primero con `find ... -print` sin `-delete`.

<!--
Fuentes consolidadas

- `/tools/scripts/pfsense/depura_ficheros.sh`
- `/tools/scripts/pfsense/pfsense_backups_auto.sh`
-->
