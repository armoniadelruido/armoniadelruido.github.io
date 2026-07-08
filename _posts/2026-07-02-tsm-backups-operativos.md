---
title: TSM y backups operativos
date: 2026-07-02 01:00:00
categories: [Storage & backups, Backups]
tags: [sysadmin, tsm, backup, restore, scheduler, veeam]
---

# TSM y backups operativos

Este post cubre comprobaciones de TSM/IBM Spectrum Protect, revision de schedules, consulta de backups y restores operativos.

## Estado del scheduler

```bash
systemctl status dsmcad
systemctl status dsmsched
ps -ef | grep -E 'dsmcad|dsmsched' | grep -v grep
```

## Logs habituales

```bash
tail -f /opt/tivoli/tsm/client/ba/bin/dsmerror.log
tail -f /opt/tivoli/tsm/client/ba/bin/dsmsched.log
```

## Consulta de backups

```bash
dsmc query backup /ruta/ejemplo/*
dsmc query filespace
dsmc query schedule
```

## Restore de fichero

```bash
dsmc restore /ruta/origen/fichero /ruta/destino/fichero
```

## Restore recursivo

```bash
dsmc restore -subdir=yes /ruta/origen/ /ruta/destino/
```

## Reinicio controlado

```bash
systemctl stop dsmsched
systemctl stop dsmcad
systemctl start dsmcad
systemctl start dsmsched
```

## SQL Server y schedules

```bash
dsmc query schedule
dsmc query event * * begindate=-1 enddate=today
```

## Protectier y cintas

```bash
dsmc query filespace
dsmc query backup /ruta/ejemplo/* -inactive
```

## Restore con fecha

```bash
dsmc restore /ruta/origen/fichero /ruta/destino/fichero -pitdate=<YYYY-MM-DD> -pittime=<HH:MM:SS>
```

<!--
Fuentes consolidadas

- `tsm.txt`
- `TSM.txt`
- `tsm (2).txt`
- `recrear_TSM.txt`
- `reiniciar TSM.txt`
- `Scheduler-tsm.txt`
- `TSM_VEEAM.txt`
- `tivol_tws.txt`
- `restore_ciclo2.txt`
- `cintas_protectier.txt`
- `protectier.txt`
- `restore_marc.txt`
- `new 3 (2).txt`
- `new 5.txt`
- `new 10.txt`
- `new 14.txt`
- `new 20.txt`
- `new 28.txt`
- `new 53.txt`
- `new 67.txt`
- `new 80.txt`
- `new 81.txt`
- `new 85.txt`
- `new 88.txt`
-->
