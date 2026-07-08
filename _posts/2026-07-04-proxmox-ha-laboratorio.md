---
title: Proxmox HA y laboratorio
date: 2026-07-04 01:00:00
categories: [Homelab, Proxmox]
tags: [proxmox, pve, ha, cluster, homelab, kubernetes]
---

# Proxmox HA y laboratorio

Este post cubre notas de laboratorio con Proxmox: comprobaciones de cluster, HA, plantillas, maquinas virtuales y pequenos entornos de pruebas.

## Estado del cluster

```bash
pvecm status
pvecm nodes
ha-manager status
```

## Recursos

```bash
qm list
pct list
pvesm status
```

## Backup de VM

```bash
vzdump <VMID> --mode snapshot --compress zstd --storage <STORAGE>
```

## Restaurar VM

```bash
qmrestore /ruta/ejemplo/vzdump-qemu-<VMID>.vma.zst <NUEVO_VMID> --storage <STORAGE>
```

## Plantilla Debian

```bash
qm template <VMID>
qm clone <TEMPLATE_ID> <NUEVO_VMID> --name <NOMBRE_VM>
```

## Proxmox Backup Client

```bash
proxmox-backup-client login
proxmox-backup-client backup tools.pxar:/tools
proxmox-backup-client backup nextcloud.pxar:/ruta/ejemplo/nextcloud
proxmox-backup-client snapshot list
```

Explorar catalogo de un snapshot:

```bash
proxmox-backup-client catalog dump host/<HOSTNAME>/<SNAPSHOT>
proxmox-backup-client catalog shell host/<HOSTNAME>/<SNAPSHOT> tools.pxar
```

Restaurar contenido:

```bash
proxmox-backup-client restore host/<HOSTNAME>/<SNAPSHOT> tools.pxar /ruta/restore/
```

Variables habituales:

```bash
export PBS_PASSWORD=<PASSWORD>
export PBS_REPOSITORY=<USUARIO>@pbs@<HOSTNAME>:<DATASTORE>
```

Excluir rutas con `.pxarexclude` dentro del directorio que no se quiera respaldar.

<!--
Fuentes consolidadas

- `proxmox`
- `Proxmox_HA.pdf`
- `Guia-Cluster-Alta-disponibilidad.pdf`
- `vdi_raspberry.txt`
- `smallab-k8s-pve-guide-main.zip`
- `/tools/scripts/proxmox_comands`
-->
