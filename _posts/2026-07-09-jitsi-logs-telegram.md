---
title: Jitsi logs y Telegram
date: 2026-07-09 00:50:00
categories: [Sistemas, Scripting]
tags: [bash, jitsi, logs, telegram, apache]
---

# Jitsi logs y Telegram

Este bloque extrae salas Jitsi desde logs Apache y envia el resumen a Telegram.

## Uso en la infraestructura

Se usa para tener visibilidad basica del uso de Jitsi sin entrar en analitica pesada.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| Auditoria ligera | Jitsi | No queda resumen rapido de salas detectadas |
| Logs web | Apache | Cambios de formato rompen el parseo |
| Avisos | Telegram | No llegan notificaciones al canal operativo |

Es util para comprobaciones diarias o alertas simples, no para metricas historicas completas.

El parseo depende del formato actual del access log. Si cambia Apache, proxy o Jitsi, campos como IP, fecha o URL pueden moverse. Antes de automatizar el envio a Telegram, revisa el fichero generado y evita mandar IPs o nombres de sala sensibles a canales compartidos. Si hay muchas lineas, mejor agrupar un resumen para no saturar el bot.

## Script original

```bash
#!/bin/bash

grep http-bind?room= /var/log/apache2/jitsi/access.log|awk '{print ($1,$2,$3,$4,$7)}'|awk -F- '{print ($1,$4)}'|awk -F? '{print ($1,$2)}' |awk '{print ($1,$3)}'|grep -v <IP_INTERNA>|sort -u > /var/log/apache2/jitsi/acces_rooms.log
```

## Bot original

```bash
#!/bin/bash

cd /var/log/nginx/jitsi/
TOKEN="<TOKEN>"
ID="<CHAT_ID>"
ID2="<CHAT_ID>"
ID_SISTEMAS="<CHAT_ID>"
MENSAJE="Esto es un Mensaje de Prueba"
URL="https://api.telegram.org/bot$TOKEN/sendMessage"

IFS=$'\n'
for j in $(cat /var/log/apache2/jitsi/acces_rooms.log)
do
    curl -s -X POST $URL -d chat_id=$ID2 -d text="$j"
done
```

## Version revisada del parseo

```bash
#!/usr/bin/env bash
set -euo pipefail

LOG_APACHE="/var/log/apache2/jitsi/access.log"
SALIDA="/var/log/apache2/jitsi/acces_rooms.log"

awk '/http-bind\?room=/ {print $1, $4, $7}' "$LOG_APACHE" \
  | awk -F- '{print $1, $4}' \
  | awk -F? '{print $1, $2}' \
  | awk '{print $1, $3}' \
  | grep -v '<IP_INTERNA>' \
  | sort -u > "$SALIDA"
```

## Version revisada del envio a Telegram

```bash
#!/usr/bin/env bash
set -euo pipefail

TOKEN="<TOKEN>"
CHAT_ID="<CHAT_ID>"
SALIDA="/var/log/apache2/jitsi/acces_rooms.log"
URL="https://api.telegram.org/bot${TOKEN}/sendMessage"

while IFS= read -r linea; do
  curl -fsS -X POST "$URL" \
    -d chat_id="$CHAT_ID" \
    --data-urlencode text="$linea"
done < "$SALIDA"
```

## Recomendaciones

- Usar `--data-urlencode` para textos con espacios o caracteres especiales.
- No guardar tokens en el script; cargarlos desde `/ruta/segura/telegram.env`.
- Añadir control de rate limit si hay muchas salas.
- Normalizar el nombre de salida `access_rooms.log`.

## Cambios y motivo

| Cambio | Motivo |
|---|---|
| `awk` directo | reduce cadena de pipes y hace el parseo mas legible |
| `while read` | evita problemas con `for $(cat fichero)` y espacios |
| `--data-urlencode` | envia correctamente texto con espacios o caracteres especiales |
| token en ruta segura | evita dejar el token de Telegram en el script |

<!--
Fuentes consolidadas

- `/tools/scripts/libra_scripts/jitsi.sh`
- `/tools/scripts/libra_scripts/jitsi1.sh`
- `/tools/scripts/libra_scripts/jitsi_live.sh`
- `/tools/scripts/libra_scripts/libra_bot.sh`
- `/tools/scripts/libra_scripts/libra_bot2.sh`
-->
