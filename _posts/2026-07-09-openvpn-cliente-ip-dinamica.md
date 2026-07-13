---
title: OpenVPN cliente con IP dinamica
date: 2026-07-09 00:40:00
categories: [Sistemas, Scripting]
tags: [bash, openvpn, exim4, vpn, ddns]
---

# OpenVPN cliente con IP dinamica

Estos scripts actualizan un perfil cliente OpenVPN cuando cambia la IP publica y lo envian por correo como adjunto.

## Uso en la infraestructura

Se usa para mantener acceso remoto al homelab cuando el cliente VPN no apunta a un FQDN estable o necesita recibir un perfil actualizado.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| Acceso remoto | OpenVPN | Los clientes quedan apuntando a una IP antigua |
| Distribucion | Correo saliente | El perfil actualizado no llega al usuario |
| Continuidad | Administracion remota | Se pierde acceso externo si no hay otra via |

Si DDNS funciona y el perfil usa FQDN, este script pasa a ser un respaldo operativo, no la pieza principal.

## Flujo

1. Obtiene la IP publica actual.
2. Lee el valor `remote` del `.ovpn`.
3. Si la IP cambia, guarda copia del perfil.
4. Sustituye la linea `remote`.
5. Envia el `.ovpn` actualizado con `exim4`.

## Script original anonimizado

```bash
#!/bin/bash
#Comparo los dos resultados, si son difefetentes meto el segundo en el fichero y restarto

LOG=/var/log/open_vpn_clients.log
FECHA=D$(date +%Y%m%d)
FROM="<EMAIL>"
TO="<EMAIL>"
SUBJECT="Soy Libra.Nueva configuracion cliente VPN"
BODY="Nueva IP: "
FILE="/ruta/segura/openvpn/openvpn-Movil-inline.ovpn"
archivo_ip="/ruta/segura/openvpn/openvpn-Movil-inline.ovpn"

ip_actual=$(dig +short myip.opendns.com @resolver1.opendns.com)
ip_guardada=$(grep 'remote' "$archivo_ip" | awk '{print $2}'|awk NR==1)

if [ "$ip_actual" != "$ip_guardada" ]; then
    cp -p "$archivo_ip" "${archivo_ip}_backup_$(date +%Y%m%d)"
    sed -i "/public/c\remote $ip_actual 1194 \# public address" "$archivo_ip"
    echo "La IP ha cambiado. El archivo ha sido actualizado y envio mail." >$LOG
(
echo "From: $FROM"
echo "To: $TO"
echo "Subject: $SUBJECT"
echo "MIME-Version: 1.0"
echo "Content-Type: multipart/mixed; boundary=\"FILEBOUNDARY\""
echo "--FILEBOUNDARY"
echo "Content-Type: text/plain; charset=UTF-8"
echo ""
echo "$BODY" $ip_actual
echo ""
echo "--FILEBOUNDARY"
echo "Content-Type: application/octet-stream; name=\"$(basename $FILE)\""
echo "Content-Disposition: attachment; filename=\"$(basename $FILE)\""
echo "Content-Transfer-Encoding: base64"
echo ""
base64 "$FILE"
echo "--FILEBOUNDARY--"
) | /usr/sbin/exim4 -t
else
    echo "Las IPs son iguales. No se requiere accion." >$LOG
fi
```

## Version revisada

```bash
#!/usr/bin/env bash
set -euo pipefail

LOG="/var/log/openvpn_clients_$(date +%Y%m%d).log"
FROM="<EMAIL>"
TO="<EMAIL>"
FILE="/ruta/segura/openvpn/cliente-inline.ovpn"
IP_PUBLICA="$(dig +short myip.opendns.com @resolver1.opendns.com)"
IP_PERFIL="$(awk '/^remote / {print $2; exit}' "$FILE")"

if [[ "$IP_PUBLICA" != "$IP_PERFIL" ]]; then
  cp -p "$FILE" "${FILE}.backup.$(date +%Y%m%d)"
  sed -i "/^remote /c\remote ${IP_PUBLICA} 1194 # public address" "$FILE"

  {
    printf 'From: %s\n' "$FROM"
    printf 'To: %s\n' "$TO"
    printf 'Subject: Nueva configuracion cliente VPN\n'
    printf 'MIME-Version: 1.0\n'
    printf 'Content-Type: multipart/mixed; boundary="FILEBOUNDARY"\n'
    printf -- '--FILEBOUNDARY\n'
    printf 'Content-Type: text/plain; charset=UTF-8\n\n'
    printf 'Nueva IP: %s\n' "$IP_PUBLICA"
    printf -- '--FILEBOUNDARY\n'
    printf 'Content-Type: application/octet-stream; name="%s"\n' "$(basename "$FILE")"
    printf 'Content-Disposition: attachment; filename="%s"\n' "$(basename "$FILE")"
    printf 'Content-Transfer-Encoding: base64\n\n'
    base64 "$FILE"
    printf -- '--FILEBOUNDARY--\n'
  } | /usr/sbin/exim4 -t
else
  printf 'La IP no ha cambiado\n' > "$LOG"
fi
```

## Recomendaciones

- Usar un FQDN estable en `remote` si DDNS ya esta operativo.
- Validar el `.ovpn` tras editarlo.
- Evitar enviar claves privadas por correo si hay alternativa.
- Usar una ruta restringida para perfiles VPN.

## Cambios y motivo

| Cambio | Motivo |
|---|---|
| buscar `^remote` | evita modificar otra linea que contenga `public` |
| `sed -i.bak` o copia previa | conserva perfil anterior si algo sale mal |
| comillas en variables | soporta rutas con espacios y evita globbing accidental |
| FQDN en vez de IP si es posible | elimina necesidad de reenviar perfil por cada cambio de IP |

<!--
Fuentes consolidadas

- `/tools/scripts/libra_scripts/open_vpn_clients.sh`
- `/tools/scripts/libra_scripts/envia_cliente_openvpn.sh`
-->
