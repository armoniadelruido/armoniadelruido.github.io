---
title: liberamemoria.sh
date: 2026-07-07 00:10:00
categories: [Sistemas, Scripting]
tags: [bash, memoria, linux, mantenimiento, cron]
---

# liberamemoria.sh

Este script comprueba memoria libre y, si baja de un umbral, fuerza la liberacion de caches del kernel Linux.

## Uso en la infraestructura

Se usa como mantenimiento preventivo en hosts con poca RAM o servicios que generan presion de cache.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| Mantenimiento RAM | Host Linux | No se libera cache en momentos de presion |
| Soporte cron | Nextcloud/libra | Puede aumentar swap o latencia del host |
| Diagnostico | Sistema base | No identifica fugas reales de memoria |

Debe verse como alivio puntual, no como sustituto de dimensionar memoria o corregir procesos consumidores.

## Ejecucion programada

```cron
@hourly /ruta/origen/scripts/liberamemoria.sh
```

## Flujo

1. Obtiene memoria libre con `free -m`.
2. Compara el valor con un umbral en MB.
3. Si la memoria libre es inferior o igual al umbral, escribe `3` en `/proc/sys/vm/drop_caches`.

## Script original anonimizado

```bash
#!/bin/bash
MEMORIA=$(free -m|grep Mem|awk '{print $4}')
LIBERAMEMORIA=sh /bin/echo 3 > /proc/sys/vm/drop_caches
#echo $MEMORIA
if [[ $MEMORIA -le 6144 ]]
then
        $LIBERAMEMORIA
#else echo $MEMORIA
fi
```

## Version revisada

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

## Cambios y motivo

| Cambio | Motivo |
|---|---|
| ejecutar `sync` y `echo` directamente | la redireccion a `/proc/sys/vm/drop_caches` no funciona bien guardada en variable |
| umbral configurable | permite adaptar el script a hosts con distinta RAM |
| log opcional | facilita saber cuantas veces se dispara |

## Variante nextcloud_scripts

En `micron_nextcloud` aparece una variante del script con umbral de `2048 MB` y log diario. El patron recomendado es mantener el umbral parametrizable:

```bash
UMBRAL_MB="${UMBRAL_MB:-2048}"
```

La tarea se ejecuta varias veces al dia desde `micron_nextcloud` para reducir presion de memoria en el host Nextcloud.

## Variante libra_scripts

La variante de `libra_scripts` usa un umbral de `512 MB`. Es util como ejemplo minimo, pero conviene ejecutar `sync` y `drop_caches` directamente en lugar de intentar guardar la redireccion en una variable.

<!--
Fuentes consolidadas

- `/tools/scripts/micron1`
- `/tools/scripts/liberamemoria.sh`
- `/tools/scripts/nextcloud_scripts/micron_nextcloud`
- `/tools/scripts/nextcloud_scripts/liberamemoria.sh`
- `/tools/scripts/libra_scripts/liberamemoria.sh`
-->
