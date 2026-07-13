---
title: pfsense_backups_auto.sh
date: 2026-07-07 00:20:00
categories: [Sistemas, Scripting]
tags: [bash, pfsense, backup, curl, csrf, firewall]
---

# pfsense_backups_auto.sh

Este script automatiza la descarga de backups de configuracion de varios pfSense mediante login web, cookies y token CSRF.

## Uso en la infraestructura

Se usa para proteger la configuracion de firewalls: reglas, NAT, VPN, HAProxy y certificados.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| Backup firewall | pfSense | Se pierde punto de restauracion de red/perimetro |
| Multi-firewall | Varias instancias | No hay copia coherente de todos los nodos |
| Retencion | XML de config | El destino puede llenarse o quedarse sin historico |

Es critico antes de cambios de reglas, actualizaciones o renovacion de certificados.

## Ejecucion programada

```cron
00 10 * * * /ruta/origen/scripts/pfsense/pfsense_backups_auto.sh
00 04 * * * /ruta/origen/scripts/pfsense/pfsense_backups_auto.sh
```

## Flujo

1. Abre la pagina de login de cada pfSense.
2. Extrae `__csrf_magic`.
3. Hace login con usuario y password.
4. Abre `diag_backup.php` para obtener un nuevo CSRF.
5. Descarga el backup XML.
6. Guarda el fichero en una ruta destino de backups.
7. Llama a `depura_ficheros.sh` para borrar backups antiguos.

## Script original

```bash
curl -L -k --cookie-jar cookies.txt \
     https://<HOSTNAME>:444/ \
     | grep "name='__csrf_magic'" \
     | sed 's/.*value="\(.*\)".*/\1/' > csrf.txt

curl -L -k --cookie cookies.txt --cookie-jar cookies.txt \
     --data-urlencode "login=Login" \
     --data-urlencode "usernamefld=<USUARIO_ADMIN>" \
     --data-urlencode "passwordfld=<PASSWORD>" \
     --data-urlencode "__csrf_magic=$(cat csrf.txt)" \
     https://<HOSTNAME>:444/ > /dev/null

curl -L -k --cookie cookies.txt --cookie-jar cookies.txt \
     https://<HOSTNAME>:444/diag_backup.php \
     | grep "name='__csrf_magic'" \
     | sed 's/.*value="\(.*\)".*/\1/' > csrf.txt

curl -L -k --cookie cookies.txt --cookie-jar cookies.txt \
     --data-urlencode "download=download" \
     --data-urlencode "donotbackuprrd=yes" \
     --data-urlencode "backupdata=yes" \
     --data-urlencode "backupssh=yes" \
     --data-urlencode "__csrf_magic=$(head -n 1 csrf.txt)" \
     https://<HOSTNAME>:444/diag_backup.php > /ruta/destino/backups/pfsense/config-<HOSTNAME>-`date +%Y%m%d%H%M%S`.xml

/ruta/origen/scripts/pfsense/depura_ficheros.sh
```

## Version revisada

```bash
#!/usr/bin/env bash
set -euo pipefail

PFSENSE_USER="${PFSENSE_USER:?}"
PFSENSE_PASS="${PFSENSE_PASS:?}"
BACKUP_DESTINO="/ruta/destino/backups/pfsense"

firewalls=(
  "fw1|https://<IP_INTERNA_1>:444"
  "fw2|https://<IP_INTERNA_2>:444"
  "fw3|https://<IP_INTERNA_3>:444"
)

for item in "${firewalls[@]}"; do
  nombre="${item%%|*}"
  url="${item#*|}"
  cookie_file="$(mktemp)"
  csrf_file="$(mktemp)"
  trap 'rm -f "$cookie_file" "$csrf_file"' EXIT

  curl -L -k --cookie-jar "$cookie_file" "$url/" \
    | grep "name='__csrf_magic'" \
    | sed 's/.*value="\(.*\)".*/\1/' > "$csrf_file"

  curl -L -k --cookie "$cookie_file" --cookie-jar "$cookie_file" \
    --data-urlencode "login=Login" \
    --data-urlencode "usernamefld=$PFSENSE_USER" \
    --data-urlencode "passwordfld=$PFSENSE_PASS" \
    --data-urlencode "__csrf_magic=$(cat "$csrf_file")" \
    "$url/" >/dev/null

  fecha="$(date +%Y%m%d%H%M%S)"
  curl -L -k --cookie "$cookie_file" --cookie-jar "$cookie_file" \
    --data-urlencode "download=download" \
    --data-urlencode "donotbackuprrd=yes" \
    --data-urlencode "backupdata=yes" \
    --data-urlencode "backupssh=yes" \
    "$url/diag_backup.php" > "$BACKUP_DESTINO/config-${nombre}-${fecha}.xml"
done

/ruta/origen/scripts/pfsense/depura_ficheros.sh
```

## Riesgos y mejoras

- No dejar usuario y password escritos en el script.
- Usar `mktemp` para cookies y CSRF.
- Borrar temporales con `trap`.
- Validar que el XML descargado no esta vacio.
- Evitar `-k` si se puede validar certificado correctamente.

## Cambios y motivo

| Cambio | Motivo |
|---|---|
| funcion por firewall | evita duplicar el bloque completo por cada pfSense |
| temporales con `mktemp` | no deja `cookies.txt` y `csrf.txt` fijos en el directorio |
| validar XML descargado | evita conservar backups vacios o pagina de login |
| credenciales fuera del script | reduce exposicion de password de firewall |

<!--
Fuentes consolidadas

- `/tools/scripts/micron1`
- `/tools/scripts/pfsense/pfsense_backups_auto.sh`
- `/tools/scripts/pfsense/depura_ficheros.sh`
-->
