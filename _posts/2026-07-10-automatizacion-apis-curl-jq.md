---
title: Automatizacion de APIs con curl y jq
date: 2026-07-10 00:40:00
categories: [Sistemas, Scripting]
tags: [curl, jq, api, bash, csv]
---

# Automatizacion de APIs con curl y jq

Los scripts revisados usan varias APIs HTTP: Cloudflare, Jira/JSM, Telegram y pfSense. El patron comun es autenticar, paginar, filtrar con `jq`, comprobar errores y generar una salida reproducible.

## Uso en la infraestructura

Este post documenta el patron comun para scripts que integran servicios externos mediante API HTTP. No representa un servicio unico, sino la base tecnica usada por automatizaciones de red, reporting, alertas y firewall.

| Uso | Servicio | Para que se usa |
|---|---|---|
| DDNS | Cloudflare | Actualizar registros DNS cuando cambia la IP publica |
| Reporting | Jira/JSM | Extraer tickets, comentarios y tiempos a CSV |
| Alertas | Telegram | Enviar avisos desde scripts de monitorizacion |
| Firewall | pfSense | Automatizar backups o acciones operativas |

Si estos patrones fallan, los scripts dependientes pueden dejar de actualizar DNS, generar informes incompletos o no enviar alertas.

## Patron recomendado: plantilla base

```bash
#!/usr/bin/env bash
set -euo pipefail

need() { command -v "$1" >/dev/null 2>&1 || { echo "Falta $1" >&2; exit 2; }; }
need curl
need jq

API_TOKEN="${API_TOKEN:-}"
[[ -z "$API_TOKEN" ]] && { echo "Falta API_TOKEN" >&2; exit 1; }
```

## Patron recomendado: GET JSON

```bash
resp="$(curl -fsS \
  -H "Authorization: Bearer <TOKEN>" \
  -H 'Accept: application/json' \
  'https://api.example.net/v1/items')"

echo "$resp" | jq -r '.items[].name'
```

## Patron recomendado: paginacion

```bash
start=0
limit=50
while :; do
  resp="$(curl -fsS --get \
    --data-urlencode "start=${start}" \
    --data-urlencode "limit=${limit}" \
    -H "Authorization: Bearer <TOKEN>" \
    'https://api.example.net/v1/items')"
  count="$(echo "$resp" | jq '.values | length')"
  (( count == 0 )) && break
  echo "$resp" | jq -c '.values[]'
  (( start += count ))
done
```

## Patron recomendado: CSV con jq

```bash
jq -r '["key","created"], (.[] | [.key, .created]) | @csv' datos.json > salida.csv
```

## Patron recomendado: fechas ISO

```bash
normalizada="$(echo "$fecha" | sed -E 's/([0-9]{2})([0-9]{2})$/\1:\2/')"
epoch="$(date -d "$normalizada" +%s)"
```

## Checklist

- Token fuera del script.
- `curl -fsS` para fallos visibles.
- `--data-urlencode` en JQL, filtros y textos.
- `mktemp` con `trap` para temporales.
- CSV generado con `jq @csv`, no concatenando comas a mano.

## Scripts originales relacionados

| Script | Uso |
|---|---|
| `jira_tool.sh` | consultar Jira/JSM y generar CSV |
| `get_jsm_keys.sh` | extraer claves JSM paginadas |
| `jira_comment_diff.sh` | calcular tiempos entre creacion y comentarios |
| `cloudflare_*.sh` | actualizar registros DNS por API |
| `libra_bot.sh` | enviar mensajes a Telegram |
| `pfsense_backups_auto.sh` | automatizar descarga de backup pfSense |

<!--
Fuentes consolidadas

- `/tools/scripts/jira_tool.sh`
- `/tools/scripts/get_jsm_keys.sh`
- `/tools/scripts/jira_comment_diff.sh`
- `/tools/scripts/jira/get_jsm_keys.sh`
- `/tools/scripts/jira/jira_tool.sh`
- `/tools/scripts/libra_scripts/cloudflare_code.sh`
- `/tools/scripts/libra_scripts/libra_bot.sh`
- `/tools/scripts/pfsense/pfsense_backups_auto.sh`
-->
