---
title: pfSense backups automaticos con curl
date: 2026-07-06 00:10:00
categories: [Sistemas, Redes]
tags: [pfsense, backup, curl, csrf, firewall, seguridad]
---

# pfSense backups automaticos con curl

Este post sirve para automatizar la descarga de backups de configuracion pfSense usando `curl`, cookies temporales y token CSRF.

## Flujo general

1. Crear ficheros temporales para cookies y respuesta HTML.
2. Obtener la pagina de login para capturar CSRF.
3. Enviar usuario, password y CSRF.
4. Descargar `diag_backup.php`.
5. Purgar backups antiguos.
6. Borrar cookies temporales aunque falle el script.

## Variables recomendadas

```bash
PFSENSE_URL="https://<IP_PFSENSE>"
PFSENSE_USER="<USUARIO_ADMIN>"
PFSENSE_PASS="<PASSWORD>"
BACKUP_DIR="/ruta/backup/pfsense"
```

## Ficheros temporales seguros

```bash
cookie_file="$(mktemp)"
login_page="$(mktemp)"
trap 'rm -f "$cookie_file" "$login_page"' EXIT
```

## Obtener CSRF

```bash
curl -k -s -c "$cookie_file" "$PFSENSE_URL/" -o "$login_page"
csrf="$(grep -oE 'name="__csrf_magic" value="[^"]+' "$login_page" | sed 's/.*value="//')"
```

## Login

```bash
curl -k -s -b "$cookie_file" -c "$cookie_file" \
  --data-urlencode "login=Login" \
  --data-urlencode "usernamefld=$PFSENSE_USER" \
  --data-urlencode "passwordfld=$PFSENSE_PASS" \
  --data-urlencode "__csrf_magic=$csrf" \
  "$PFSENSE_URL/" >/dev/null
```

## Descargar backup

```bash
fecha="$(date +%Y%m%d_%H%M%S)"
curl -k -s -b "$cookie_file" \
  "$PFSENSE_URL/diag_backup.php?download=download" \
  -o "$BACKUP_DIR/pfsense_${fecha}.xml"
```

## Rotacion

```bash
find "$BACKUP_DIR" -type f -name 'pfsense_*.xml' -mtime +15 -delete
```

## Recomendaciones

- Usar un usuario dedicado para backup.
- Guardar credenciales fuera del script, por ejemplo en `.env` con permisos `600`.
- Evitar persistir cookies o tokens CSRF.
- Validar periodicamente que el XML descargado no esta vacio.
- Probar restauracion en entorno controlado antes de confiar en la copia.

## Scripts relacionados

- `pfsense_backups_auto.sh`: automatiza login, descarga de XML y llamada a depuracion.
- `depura_ficheros.sh`: elimina backups antiguos en rutas destino.
- `pfsense-import-certificate.php`: importa certificados desde CLI, actualiza GUI/HAProxy y reinicia servicios asociados.

<!--
Fuentes consolidadas

- `/tools/scripts/pfsense/pfsense_backups_auto.sh`
- `/tools/scripts/pfsense/depura_ficheros.sh`
-->
