---
title: depura_ficheros.sh
date: 2026-07-07 00:30:00
categories: [Sistemas, Scripting]
tags: [bash, limpieza, backups, find, pfsense]
---

# depura_ficheros.sh

Este script elimina backups antiguos de varias carpetas destino de pfSense.

## Uso en la infraestructura

Se usa para aplicar retencion a copias de configuracion de firewall y evitar que el destino de backups crezca sin control.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| Retencion | Backups pfSense | El disco de backups puede llenarse |
| Higiene | XML antiguos | Se acumulan copias sin valor operativo |
| Control de riesgo | Firewall | Un patron incorrecto puede borrar mas de lo esperado |

Debe probarse primero con `find ... -print` antes de usar borrado.

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

## Script original

```bash
#!/bin/bash
LOG=/var/log/depura_pfsense.log
FECHA=D$(date +%Y%m%d)
sz_ORIONAD='/ruta/destino/backups/pfsense/fw1'
sz_PFSENSE='/ruta/destino/backups/pfsense/fw2'
sz_PFSENSE2='/ruta/destino/backups/pfsense/fw3'

echo "Depuro carpeta ORIONAD" >> $LOG
find $sz_ORIONAD -mtime +15 -exec rm {} \;
if [ $? -ne 0 ]; then echo "Error al depurar ORIONAD" >> $LOG; exit 5; else
echo "Depuro carpeta PFSENSE" >> $LOG
find $sz_PFSENSE -mtime +15 -exec rm {} \;
if [ $? -ne 0 ]; then echo "Error al depurar PFSENSE" >> $LOG; exit 7; else
echo "Depuro carpeta PFSENSE2" >> $LOG
find $sz_PFSENSE2 -mtime +15 -exec rm {} \;
if [ $? -ne 0 ]; then echo "Error al depurar PFSENSE2" >> $LOG; exit 9; else
echo "Proceso finalizado correctamente"
exit 0
fi; fi; fi
exit 0
```

## Version revisada

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

## Cambios y motivo

| Cambio | Motivo |
|---|---|
| array de directorios | evita repetir el mismo bloque por cada firewall |
| `-type f` | no borra directorios por accidente |
| `-name '*.xml'` | limita la limpieza a backups pfSense |
| `-print -delete` | deja evidencia de lo eliminado |

<!--
Fuentes consolidadas

- `/tools/scripts/pfsense/depura_ficheros.sh`
- `/tools/scripts/pfsense/pfsense_backups_auto.sh`
-->
