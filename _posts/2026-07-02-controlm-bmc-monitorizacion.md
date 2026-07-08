---
title: Control-M BMC y monitorizacion
date: 2026-07-02 01:20:00
categories: [Sistemas, Operación]
tags: [sysadmin, control-m, bmc, nagios, monitoring, operacion]
---

# Control-M BMC y monitorizacion

Este post ayuda a revisar agentes Control-M, validar monitorizacion, comprobar Nagios y documentar tareas de contingencia operacional.

## Control-M: comprobaciones basicas

```bash
ctmpsm
ctm_menu
ctmipc -dest all -msg status
```

## Reinicio controlado de agente

```bash
ctm_agstat
ctm_agstop
ctm_agstart
ctm_agstat
```

## Nagios: servicio y configuracion

```bash
systemctl status nagios
nagios -v /etc/nagios/nagios.cfg
systemctl reload nagios
```

## Logs de monitorizacion

```bash
tail -f /var/log/nagios/nagios.log
journalctl -u nagios -f
```

## Migraciones y contingencia

```bash
ctmpsm
ctmipc -dest all -msg status
ctm_agstat
```

Registrar siempre ventana, servidor origen, servidor destino, jobs afectados y plan de vuelta atras.

## Checks de filesystem

```bash
df -h
/usr/lib/nagios/plugins/check_disk -w 20% -c 10% -p /ruta/ejemplo
```

<!--
Fuentes consolidadas

- `CONTROL-M.txt`
- `controlM.txt`
- `CONTROL-M-Destartarlo.txt`
- `CONTROL-M_ASVs.txt`
- `ControlM_Migracion.txt`
- `controlm_contingencia.txt`
- `bladelogic.txt`
- `Truesight.txt`
- `Tarea_BMC_Agentes_root.txt`
- `nagios_install.txt`
- `nagios_weblogic.txt`
- `control_certis_nagios.txt`
- `Controls_Mati.txt`
- `bsa_cosas.txt`
- `mail_bmc.txt`
- `inventario_nagios.txt`
- `control_FS_nagios_aix.txt`
- `nagios_aws.txt`
- `new 2.txt`
- `new 4.txt`
- `new 39.txt`
-->
