---
title: NetApp y snapshots
date: 2026-07-02 01:10:00
categories: [Storage & backups, NetApp]
tags: [sysadmin, netapp, storage, snapshots, cabinas]
---

# NetApp y snapshots

Este post sirve para revisar estado de cabinas NetApp, consultar espacio, gestionar snapshots y preparar restauraciones puntuales.

Antes de borrar o restaurar snapshots, confirma SVM, volumen y snapshot exacto. Siempre que puedas, prefiere restaurar un fichero concreto antes que revertir un volumen entero: `snapshot restore` puede volver atras datos que otros usuarios han cambiado despues del snapshot.

## Comprobaciones generales

```bash
system health status show
cluster show
node show
storage aggregate show
volume show
```

## Espacio

```bash
df -h
volume show -fields size,available,used,percent-used
aggregate show-space
```

## Snapshots

```bash
snapshot show -vserver <SVM> -volume <VOLUME>
snapshot create -vserver <SVM> -volume <VOLUME> -snapshot <SNAPSHOT>
snapshot delete -vserver <SVM> -volume <VOLUME> -snapshot <SNAPSHOT>
```

## Restaurar desde snapshot

```bash
snapshot restore-file \
  -vserver <SVM> \
  -volume <VOLUME> \
  -snapshot <SNAPSHOT> \
  -path /ruta/ejemplo/fichero \
  -restore-path /ruta/ejemplo/fichero.restore
```

## Restaurar snapshot completo

```bash
snapshot restore -vserver <SVM> -volume <VOLUME> -snapshot <SNAPSHOT>
```

## Pure Storage relacionado

```bash
purevol list
purevol snap <VOLUME>
purevol copy <SNAPSHOT> <VOLUME_DESTINO>
```

<!--
Fuentes consolidadas

- `Netapp.txt`
- `disco_netapp.txt`
- `reactivacion_snapshots_cabina.txt`
- `Intervencion_Cabina.txt`
- `protectier.txt`
- `restaura_snaps.txt`
- `comandos_pure.txt`
- `new 37.txt`
- `new 108.txt`
-->
