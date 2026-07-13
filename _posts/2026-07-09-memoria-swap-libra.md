---
title: Memoria y swap en libra_scripts
date: 2026-07-09 01:10:00
categories: [Sistemas, Scripting]
tags: [bash, memoria, swap, linux, mantenimiento]
---

# Memoria y swap en libra_scripts

Este bloque agrupa mantenimiento basico de memoria: liberar caches cuando queda poca RAM, apagar swap concreta y revisar consumo por proceso.

## Uso en la infraestructura

Se usa como mantenimiento puntual en hosts pequenos que ejecutan servicios web, Nextcloud, Jitsi o tareas de backup.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| Alivio temporal | Kernel/Linux | El host puede seguir con presion de memoria |
| Swap | Sistema base | Un `swapoff` mal aplicado puede dejar procesos sin memoria |
| Diagnostico | Procesos | No se identifica el consumidor real |

No sustituye ampliar RAM, corregir fugas o ajustar servicios; solo ayuda a operaciones puntuales.

## Script original de memoria anonimizado

```bash
#!/bin/bash
MEMORIA=$(free -m|grep Mem|awk '{print $4}')
LIBERAMEMORIA=sh /usr/bin/echo 3 > /proc/sys/vm/drop_caches
if [[ $MEMORIA -le 512 ]]
then
        $LIBERAMEMORIA
else
        exit 0
fi
```

## Script original de swap anonimizado

```bash
#!/bin/bash
#listo la swap y parseo la columna y la linea
szKK=`cat /proc/swaps |awk '{print $1}'|awk 'NR==3'`
echo "libero" $szKK
swapoff $szKK
exit 0
```

## Version revisada de liberar caches

```bash
#!/usr/bin/env bash
set -euo pipefail

UMBRAL_MB="${UMBRAL_MB:-512}"
MEMORIA_LIBRE="$(free -m | awk '/^Mem:/ {print $4}')"

if [[ "$MEMORIA_LIBRE" -le "$UMBRAL_MB" ]]; then
  sync
  echo 3 > /proc/sys/vm/drop_caches
fi
```

## Version revisada de apagar swap concreta

```bash
#!/usr/bin/env bash
set -euo pipefail

SWAP_DEVICE="$(awk 'NR==3 {print $1}' /proc/swaps)"

if [[ -n "$SWAP_DEVICE" ]]; then
  swapoff "$SWAP_DEVICE"
fi
```

## Recomendaciones

- No usar `drop_caches` como solucion a fugas de memoria.
- Revisar procesos con consumo alto antes de liberar caches.
- Confirmar espacio libre en RAM antes de ejecutar `swapoff`.
- Evitar depender de una linea fija de `/proc/swaps` si hay varias swaps.

## Cambios y motivo

| Cambio | Motivo |
|---|---|
| ejecutar `sync` y `echo` directamente | la redireccion no queda bien guardada dentro de una variable |
| umbral parametrizable | permite usar 512 MB o 2048 MB segun host |
| comillas en `$SWAP_DEVICE` | evita errores si la variable queda vacia |
| no depender de `NR==3` | la swap activa puede no estar siempre en esa linea |

<!--
Fuentes consolidadas

- `/tools/scripts/libra_scripts/liberamemoria.sh`
- `/tools/scripts/libra_scripts/libera_swap.sh`
- `/tools/scripts/libra_scripts/ps_mem.py`
-->
