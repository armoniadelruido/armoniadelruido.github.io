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

<!--
Fuentes consolidadas

- `proxmox`
- `Proxmox_HA.pdf`
- `Guia-Cluster-Alta-disponibilidad.pdf`
- `vdi_raspberry.txt`
- `smallab-k8s-pve-guide-main.zip`
-->
