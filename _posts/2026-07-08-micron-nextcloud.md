---
title: micron_nextcloud
date: 2026-07-08 00:10:00
categories: [Sistemas, Scripting]
tags: [bash, cron, nextcloud, backup, mantenimiento]
---

# micron_nextcloud

Este fichero actua como crontab de mantenimiento para Nextcloud: libera memoria, lanza backups locales, dumpea la BBDD y mueve copias hacia almacenamiento externo.

## Uso en la infraestructura

Es el planificador central de mantenimiento del host Nextcloud. Coordina tareas de memoria, backups, dumps SQL y movimiento de copias.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| Cron operativo | Nextcloud | No se ejecutan backups ni mantenimiento periódico |
| Orquestacion | Scripts de backup | Se rompe la cadena SQL, filesystem y retencion |
| Control horario | Host Nextcloud | Tareas pesadas pueden solaparse o no lanzarse |

Debe revisarse cuando se cambia una ruta, se mueve un script o se modifica la ventana de backups.

## Crontab original

```cron
0 0,9,10,12,14,16,18,20,22,23 * * * /ruta/origen/scripts/liberamemoria.sh
30 9 * * * /ruta/origen/scripts/cirro_bot_espacio.sh
30 1 * * 1-5 /ruta/origen/scripts/backup_filesystem.sh
10 0 * * * /ruta/origen/scripts/dumpea.sh
15 3 * * 0 /ruta/origen/scripts/muevo_sqls.sh
30 2 1 * * /ruta/origen/scripts/muevo_filesystems_neptuno.sh
```

## Entradas comentadas relevantes

```cron
#30 3 * * 1-5 /ruta/origen/scripts/sincro_nubes.sh
#15 3 * * 0 /ruta/origen/scripts/muevo_tools_etc_urano.sh
#30 4 1 * * /ruta/origen/scripts/backup_nube_neptuno.sh
```

## Version revisada

```cron
0 0,9,10,12,14,16,18,20,22,23 * * * /ruta/origen/scripts/liberamemoria.sh
30 1 * * 1-5 /ruta/origen/scripts/backup_filesystem.sh
10 0 * * * /ruta/origen/scripts/dumpea.sh
15 3 * * 0 /ruta/origen/scripts/muevo_sqls.sh
30 2 1 * * /ruta/origen/scripts/muevo_filesystems_neptuno.sh
```

## Cambios y motivo

| Cambio | Motivo |
|---|---|
| separar activos y comentados | diferencia operativa actual de tareas historicas |
| revisar rutas absolutas | el cron referencia una ubicacion distinta a la carpeta analizada |
| evitar solapes horarios | backups SQL, filesystem y movimiento no deben pisarse |
| documentar script no encontrado | `cirro_bot_espacio.sh` queda como dependencia externa |

## Grafo de llamadas

```text
micron_nextcloud
├── liberamemoria.sh
├── cirro_bot_espacio.sh          # no encontrado localmente
├── backup_filesystem.sh
├── dumpea.sh
├── muevo_sqls.sh
└── muevo_filesystems_neptuno.sh
    └── depuro_neptuno_90.sh
```

Scripts comentados relacionados:

```text
micron_nextcloud
├── sincro_nubes.sh
│   └── import_sql.sh
├── muevo_tools_etc_urano.sh
└── backup_nube_neptuno.sh
```

## Observaciones

- El crontab referencia `/ruta/origen/scripts/...`, pero los ficheros analizados estaban agrupados en una subcarpeta de scripts Nextcloud.
- Conviene normalizar rutas absolutas o usar una variable `SCRIPT_DIR`.
- Las tareas de backup y movimiento comparten el patron `logline()` para timestamp en logs.
- Hay tareas comentadas que conviene mantener documentadas porque forman parte de la operativa historica.

<!--
Fuentes consolidadas

- `/tools/scripts/nextcloud_scripts/micron_nextcloud`
-->
