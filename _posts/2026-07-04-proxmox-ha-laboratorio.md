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

## Backup de configuracion PVE

```bash
fecha="$(date +%Y%m%d_%H%M%S)"
destino="/backup/pve-config-${fecha}.tar.gz"

tar czf "$destino" \
  /etc/pve \
  /etc/network/interfaces \
  /etc/hosts \
  /etc/cron* \
  /etc/apcupsd 2>/dev/null
```

## Password file y fingerprint

```bash
export PBS_PASSWORD_FILE="/ruta/segura/pbs_password"
proxmox-backup-client backup etc.pxar:/etc \
  --repository <USUARIO>@pbs@<HOSTNAME>:<DATASTORE> \
  --fingerprint <FINGERPRINT>
```

## Comentarios via API PBS

Preferir API token o cookie temporal generada en runtime:

```bash
curl -k -X PUT \
  -H "Authorization: PBSAPIToken=<TOKEN>" \
  -H "Content-Type: application/json" \
  "https://<PBS_HOST>:8007/api2/json/admin/datastore/<DATASTORE>/snapshots/<SNAPSHOT>/notes" \
  -d '{"notes":"Backup validado"}'
```

## Script relacionado

`proxmox_backup_collar.sh` usa `proxmox-backup-client` para respaldar varias rutas origen hacia un repositorio PBS y despues invoca una limpieza de memoria en el host remoto.

<!--
Fuentes consolidadas

- `proxmox`
- `Proxmox_HA.pdf`
- `Guia-Cluster-Alta-disponibilidad.pdf`
- `vdi_raspberry.txt`
- `smallab-k8s-pve-guide-main.zip`
- `/tools/scripts/proxmox_comands`
- `/tools/scripts/proxmox_backup_collar.sh`
- `/tools/scripts/proxmox_backup_filesystem_ORIG`
- `/tools/scripts/proxmox-pve-config-backup.sh`
- `/tools/scripts/edito_comment.sh`
-->
