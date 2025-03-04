---
title: Importar disco de Vmware a Proxmox
date: 2025-02-12 0:31:00
categories: [Homelab, Proxmox]
tags: [sysdamin, shell, bash, scripting, proxmox, qm, vmdk, qcow2 ]
---

### Cositas de proxmox:

Como convertir un disco de vmare ova, a un disco que proxmox pueda usar:

```bash
tar xvf /tmp/VMware-Workstation.ova
qemu-img convert -f vmdk -O qcow2 VMware-Workstation-0.vmdk /mnt/pve/storage01/images/406/vm-406-disk-0.qcow2
qm set 406 --scsi0 storage01:406/vm-406-disk-0.qcow2

```
