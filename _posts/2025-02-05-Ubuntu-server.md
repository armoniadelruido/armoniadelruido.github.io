---
title: Ubuntu
date: 2025-02-01 0:31:00
categories: [Sistemas, Ubuntu]
tags: [sysdamin, ubuntu, netplan, ip, ss]
---

## Cuando instalamos Ubuntu Server minimal, nos encontramos con que no tiene asignada la parametrización de la interfaz de red.
Se configura con netplan:
```bash
vi /etc/netplan/00-installer-config.yaml


# This is the network config written by 'subiquity'
network:
  ethernets:
   enp6s18:
    dhcp4: true
  version: 2
```

Y luego aplicamos:
```bash
netplan try
netplan apply
```
Podriamos querer que en lugar de asignacion por dhcp, fuera ip estatica, lo pondriamos asi:

```bash
vi /etc/netplan/00-installer-config.yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
   enp6s18:
    addresses: [192.168.10.12/24]
    gateway4: 192.168.10.1
    nameservers:
     search: [redinterna.local]
     addresses: [192.168.10.10]
version: 2
```
