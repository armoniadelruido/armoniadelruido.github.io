---
title: APC UPS con apcupsd
date: 2026-07-04 00:30:00
categories: [Sistemas, Homelab]
tags: [sysadmin, ups, apcupsd, energia, monitorizacion]
---

# APC UPS con apcupsd

Este post ayuda a instalar `apcupsd`, comprobar el estado de un SAI APC y preparar acciones basicas ante cortes electricos.

## Instalacion

```bash
apt install -y apcupsd
```

## Configuracion minima

Editar `/etc/apcupsd/apcupsd.conf`:

```text
UPSCABLE usb
UPSTYPE usb
DEVICE
ONBATTERYDELAY 6
BATTERYLEVEL 5
MINUTES 3
TIMEOUT 0
```

Activar servicio en `/etc/default/apcupsd`:

```text
ISCONFIGURED=yes
```

## Servicio

```bash
systemctl enable apcupsd
systemctl restart apcupsd
systemctl status apcupsd
```

## Estado del SAI

```bash
apcaccess status
```

<!--
Fuentes consolidadas

- `apcupsd-UPS`
-->
