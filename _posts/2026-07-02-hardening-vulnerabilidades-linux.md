---
title: Hardening y vulnerabilidades Linux
date: 2026-07-02 00:40:00
categories: [Sistemas, Seguridad]
tags: [sysadmin, hardening, seguridad, firewall, iptables, vulnerabilidades]
---

# Hardening y vulnerabilidades Linux

Este post ayuda a revisar el estado de seguridad de servidores Linux, comprobar parches, servicios expuestos, accesos, firewall y logs relevantes.

## Identificacion del sistema

```bash
hostnamectl
cat /etc/os-release
uname -a
rpm -qa --last | head
```

## Parches y repositorios

```bash
yum check-update
yum updateinfo list security all
yum updateinfo list cves
```

## Servicios expuestos

```bash
ss -tulpen
systemctl list-unit-files --state=enabled
systemctl --failed
```

## Usuarios y accesos

```bash
getent passwd
awk -F: '($3 == 0) {print $1}' /etc/passwd
lastlog
faillock --user <USUARIO>
```

## SSH

Revisar al menos:

```text
PermitRootLogin no
PasswordAuthentication no
Protocol 2
AllowUsers <USUARIO>
```

## Firewall

```bash
iptables -S
iptables -L -n -v
firewall-cmd --list-all
firewall-cmd --list-services
```

## SACK y kernel

```bash
sysctl net.ipv4.tcp_sack
sysctl net.ipv4.tcp_dsack
sysctl net.ipv4.tcp_fack
```

Ejemplo de mitigacion temporal si aplica:

```bash
sysctl -w net.ipv4.tcp_sack=0
```

## Logs utiles

```bash
journalctl -p warning..alert
grep -i "failed password" /var/log/secure*
ausearch -m USER_LOGIN,USER_AUTH
```

<!--
Fuentes consolidadas

- `ChecklistHardening.txt`
- `hardening_maquinas.txt`
- `intervnecion hardening.txt`
- `vulnerabilidades.txt`
- `ASVs_1T2019.txt`
- `digifest.txt`
- `digifest139.txt`
- `digifest144.txt`
- `SACK.txt`
- `Sack_panick_info.txt`
- `IP_tables.txt`
- `REGLAS FW.txt`
- `ossim_ossec.txt`
- `OSSIM_SYSLOG.txt`
- `sophos_inst.txt`
- `Talpa_sophos.txt`
- `imperva.txt`
- `Imperva_register.txt`
- `FOCA_results.txt`
-->
