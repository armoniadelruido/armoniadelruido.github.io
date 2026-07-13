---
title: Certificados Let's Encrypt pfSense y servicios
date: 2026-07-09 00:30:00
categories: [Sistemas, Scripting]
tags: [bash, letsencrypt, certbot, pfsense, haproxy]
---

# Certificados Let's Encrypt pfSense y servicios

Este bloque automatiza la renovacion de certificados wildcard con validacion DNS-01, copia los PEM a servicios internos y permite importarlos en pfSense.

## Uso en la infraestructura

Se usa para mantener TLS valido en servicios publicados y en el frontal pfSense/HAProxy.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| Emision | Let's Encrypt | No se renueva el wildcard |
| Distribucion | Nextcloud, Docs, Jitsi | Servicios con certificado caducado o antiguo |
| Frontal | pfSense/HAProxy | El certificado nuevo no queda activo en entrada |

Depende del token DNS de Cloudflare, de permisos sobre claves privadas y de conectividad SSH hacia los hosts consumidores.

## Script original anonimizado

```bash
#!/bine/bash
#LANZA ESTO: certbot certonly --dns-cloudflare --dns-cloudflare-credentials ~/.secrets/certbot/cloudflare.ini -d example.net -d *.example.net
#PROBAMOS CON NON INTERACTIVE: certbot certonly --dns-cloudflare --dns-cloudflare-credentials ~/.secrets/certbot/cloudflare.ini -n -d example.net -d *.example.net
#Y LUEGO ESTO: cp /etc/letsencrypt/live/example.net/privkey.pem /tmp/
#Y LUEGO ESTO: cp /etc/letsencrypt/live/example.net/fullchain.pem /tmp/
#Y LUEGO ESTO: chown <USUARIO>.<USUARIO> /tmp/*.pem
szLOG=/var/log/renueva_certis.log
FICHERO1=/etc/letsencrypt/live/example.net/privkey.pem
FICHERO2=/etc/letsencrypt/live/example.net/fullchain.pem

certbot certonly --dns-cloudflare --dns-cloudflare-credentials ~/.secrets/certbot/cloudflare.ini -n -d example.net -d *.example.net > $szLOG
cp $FICHERO1 $FICHERO2 /tmp/
chown <USUARIO>.<USUARIO> /tmp/*.pem
su <USUARIO> -c "/ruta/origen/scripts/certis_docserver.sh
exit
```

## Version revisada de renovacion wildcard

```bash
#!/usr/bin/env bash
set -euo pipefail

LOG="/var/log/renueva_certis_$(date +%Y%m%d%H%M).log"
DOMINIO="example.net"
CREDENCIALES_CF="/ruta/segura/cloudflare.ini"
LIVE_DIR="/etc/letsencrypt/live/${DOMINIO}"

certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials "$CREDENCIALES_CF" \
  --non-interactive \
  -d "$DOMINIO" \
  -d "*.${DOMINIO}" > "$LOG" 2>&1

install -m 600 "$LIVE_DIR/privkey.pem" /ruta/destino/certs/privkey.pem
install -m 644 "$LIVE_DIR/fullchain.pem" /ruta/destino/certs/fullchain.pem
```

## Distribucion a servicios

```bash
scp /ruta/destino/certs/privkey.pem /ruta/destino/certs/fullchain.pem \
  <USUARIO>@<HOSTNAME>:/ruta/destino/certs/

ssh <USUARIO>@<HOSTNAME> "/ruta/origen/scripts/renova.sh"
```

Este patron aparece para servicios como Nextcloud, servidor documental y Jitsi.

## Importacion en pfSense

El script PHP importa un certificado en pfSense desde CLI y despues lo asigna a servicios:

- Valida que certificado y clave privada no esten vacios.
- Comprueba que la clave corresponde al certificado.
- Importa el certificado en el almacen de pfSense.
- Lo establece como certificado de la GUI.
- Actualiza frontends HAProxy que usaban el certificado anterior.
- Reinicia servicios que dependan del certificado.

## Ejemplo de envio

```bash
scp -P 2222 \
  /ruta/origen/scripts/pfsense-import-certificate.php \
  /ruta/destino/certs/fullchain.pem \
  /ruta/destino/certs/privkey.pem \
  <USUARIO_ADMIN>@<HOSTNAME>:/ruta/destino/ssl/

ssh -p 2222 <USUARIO_ADMIN>@<HOSTNAME> \
  'php /ruta/destino/ssl/pfsense-import-certificate.php /ruta/destino/ssl/fullchain.pem /ruta/destino/ssl/privkey.pem'
```

## Riesgos y mejoras

- Probar `certbot renew --dry-run` antes de automatizar.
- No copiar claves privadas a `/tmp` si se puede usar una ruta segura.
- Validar que el certificado nuevo no caduca antes que el actual.
- Registrar checksum y fecha de expiracion tras cada despliegue.

## Cambios y motivo

| Cambio | Motivo |
|---|---|
| corregir shebang | `#!/bine/bash` no apunta al interprete correcto |
| evitar `/tmp` para claves | reduce exposicion de `privkey.pem` |
| `install -m` | copia y aplica permisos en el mismo paso |
| separar renovacion y distribucion | permite validar certificado antes de desplegarlo |

<!--
Fuentes consolidadas

- `/tools/scripts/libra_scripts/renueva_certis.sh`
- `/tools/scripts/libra_scripts/letsencrypt_wildcard`
- `/tools/scripts/libra_scripts/pfsense_certs.sh`
- `/tools/scripts/libra_scripts/piholes_certs.sh`
- `/tools/scripts/libra_scripts/certis_nextcloud.sh`
- `/tools/scripts/libra_scripts/certis_docserver.sh`
- `/tools/scripts/libra_scripts/certis_jitsiking.sh`
- `/tools/scripts/libra_scripts/pfsense-import-certificate.php`
-->
