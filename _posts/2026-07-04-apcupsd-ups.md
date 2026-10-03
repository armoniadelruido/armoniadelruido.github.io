---
title: APC UPS con apcupsd
date: 2026-07-04 00:30:00
categories: [Sistemas, Homelab]
tags: [sysadmin, ups, apcupsd, energia, monitorizacion]
---

# APC UPS con apcupsd

`apcupsd` permite que un equipo conectado por USB a un SAI comparta su estado con otros servidores. Así, los clientes pueden apagarse antes que el equipo maestro cuando la batería baja.

En los ejemplos no se publican nombres ni direcciones reales. Sustituye `<IP_INTERNA_MASTER>` únicamente en la copia privada de tu configuración.

## Esquema

```text
SAI APC --USB--> MASTER --TCP/3551--> CLIENTE 1
                              └-----> CLIENTE 2
```

- **Master**: tiene el cable USB y publica el estado mediante NIS.
- **Clientes**: consultan al master y aplican sus propios umbrales de apagado.
- Los clientes deben apagarse antes que el master para conservar batería y permitir un cierre ordenado.

## Instalacion

En todos los equipos:

```bash
sudo apt update
sudo apt install apcupsd
sudo cp /etc/apcupsd/apcupsd.conf /etc/apcupsd/apcupsd.conf.bak
```

Activar el servicio en `/etc/default/apcupsd`:

```text
ISCONFIGURED=yes
```

## Configuración del master

Valores principales de `/etc/apcupsd/apcupsd.conf`:

```text
UPSCABLE usb
UPSTYPE usb
DEVICE
POLLTIME 60

ONBATTERYDELAY 6
BATTERYLEVEL 30
MINUTES 10
TIMEOUT 0

NETSERVER on
NISIP <IP_INTERNA_MASTER>
NISPORT 3551
```

Con un SAI USB, `DEVICE` puede quedar vacío para que `apcupsd` detecte el dispositivo. Es preferible que `NISIP` escuche solo en la interfaz interna. Si se usa `0.0.0.0`, el cortafuegos debe permitir TCP/3551 exclusivamente desde las direcciones de los clientes.

## Configuración de los clientes

En cada cliente:

```text
UPSCABLE ether
UPSTYPE net
DEVICE <IP_INTERNA_MASTER>:3551
POLLTIME 10

BATTERYLEVEL 40
MINUTES 15
TIMEOUT 0
```

Los umbrales del ejemplo hacen que los clientes inicien el apagado antes que el master. Hay que adaptarlos a la autonomía real, al consumo y al tiempo que necesita cada sistema para detener servicios.

## Arranque y comprobación

```bash
sudo systemctl enable apcupsd
sudo systemctl restart apcupsd
sudo systemctl status apcupsd
apcaccess status
```

Desde un cliente también se puede consultar explícitamente el servidor:

```bash
apcaccess status <IP_INTERNA_MASTER>:3551
```

Los campos más útiles son:

- `STATUS`: estado de red o batería.
- `BCHARGE`: porcentaje de carga.
- `TIMELEFT`: autonomía estimada.
- `LOADPCT`: carga conectada.
- `LASTXFER`: causa del último cambio a batería.

## Logs y conectividad

```bash
journalctl -u apcupsd -f
tail -f /var/log/apcupsd.events
ss -lnt | grep 3551
```

Si el cliente no recibe información, comprobar en este orden:

1. El master detecta el SAI con `apcaccess status`.
2. `NETSERVER` está activo y escucha en la IP esperada.
3. TCP/3551 está permitido solo entre clientes y master.
4. `DEVICE` en el cliente apunta a la dirección correcta.
5. Los relojes de los equipos están sincronizados para interpretar los eventos.

## Prueba controlada

Antes de depender del sistema conviene hacer una prueba en una ventana de mantenimiento:

1. Confirmar que todos los equipos ven el mismo estado del SAI.
2. Desconectar la alimentación de entrada del SAI, sin desconectar las cargas.
3. Verificar el cambio a `ONBATT` y la aparición del evento en los logs.
4. Reconectar antes de alcanzar los umbrales si solo se valida la monitorización.
5. Hacer una prueba completa de apagado cuando exista consola remota y un plan de recuperación.

No se debe simular el corte desenchufando el cable USB: eso prueba la pérdida de comunicación, no un fallo eléctrico.

## Referencia

- [Manual de `apcupsd.conf`](https://manpages.debian.org/trixie/apcupsd/apcupsd.conf.5.en.html)
