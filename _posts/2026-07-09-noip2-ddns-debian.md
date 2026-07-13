---
title: noip2 DDNS en Debian
date: 2026-07-09 01:20:00
categories: [Sistemas, Scripting]
tags: [linux, debian, ddns, noip, init]
---

# noip2 DDNS en Debian

`noip2` es un cliente de DNS dinamico que mantiene actualizado un host en No-IP. El material revisado incluye notas de uso y un script init para Debian.

## Uso en la infraestructura

Se documenta como alternativa o pieza historica de DDNS para hosts Debian.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| DNS dinamico | No-IP | El hostname externo apunta a una IP antigua |
| Arranque | init Debian | El daemon no levanta tras reinicio |
| Migracion | DDNS heredado | Sirve para comparar con la operativa Cloudflare |

En la infraestructura actual encaja como referencia de DDNS clasico frente al enfoque por API de Cloudflare.

Usa estos comandos solo en sistemas donde `noip2` se haya instalado manualmente en `/usr/local/bin`. En Debian moderno suele ser mas limpio crear una unidad `systemd`; el script init queda para instalaciones antiguas. Despues de lanzar el daemon, comprueba con `noip2 -S` que solo hay una instancia activa y que el host configurado es el correcto.

## Comandos utiles

```bash
/usr/local/bin/noip2 -C       # crear configuracion
/usr/local/bin/noip2          # lanzar daemon
/usr/local/bin/noip2 -S       # ver instancias/configuracion
/usr/local/bin/noip2 -K <PID> # parar instancia
```

## Servicio init clasico

```sh
#!/bin/sh
DAEMON=/usr/local/bin/noip2
NAME=noip2

test -x "$DAEMON" || exit 0

case "$1" in
  start)
    start-stop-daemon --start --exec "$DAEMON"
    ;;
  stop)
    start-stop-daemon --stop --oknodo --retry 30 --exec "$DAEMON"
    ;;
  restart)
    start-stop-daemon --stop --oknodo --retry 30 --exec "$DAEMON"
    start-stop-daemon --start --exec "$DAEMON"
    ;;
  *)
    echo "Usage: $0 {start|stop|restart}"
    exit 1
    ;;
esac
```

## Permisos recomendados

```bash
chmod 700 /usr/local/bin/noip2
chown root:root /usr/local/bin/noip2
chmod 600 /usr/local/etc/no-ip2.conf
chown root:root /usr/local/etc/no-ip2.conf
```

## Recomendaciones

- Migrar a unidad `systemd` si el sistema ya no usa init clasico.
- Proteger `/usr/local/etc/no-ip2.conf` porque contiene credenciales.
- Evitar multiples instancias salvo que haya configuraciones separadas.

<!--
Fuentes consolidadas

- `/tools/scripts/libra_scripts/LEEME.PRIMERO`
- `/tools/scripts/libra_scripts/debian.noip2.sh`
-->
