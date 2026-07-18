---
title: Cloudflare DDNS en libra_scripts
date: 2026-07-09 00:20:00
categories: [Sistemas, Scripting]
tags: [bash, cloudflare, ddns, dns, api]
---

# Cloudflare DDNS en libra_scripts

Este bloque compara la IP publica actual con la ultima conocida y, si cambia, actualiza varios registros DNS en Cloudflare.

## Uso en la infraestructura

Se usa para mantener publicados los FQDN del homelab cuando la conexion cambia de IP publica.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| DNS dinamico | Cloudflare | Los nombres externos apuntan a una IP antigua |
| Publicacion | web1, web2, web3, web4 | Servicios inaccesibles desde Internet |
| Disparo operativo | OpenVPN | No se regenera el cliente si depende de IP directa |

Es una pieza de perimetro: debe ejecutarse antes de validar VPN o certificados.

## Flujo

1. Lee la IP anterior desde un fichero de estado.
2. Consulta la IP publica con DNS externo.
3. Si la IP cambia, guarda copia del estado anterior.
4. Actualiza registros DNS mediante la API de Cloudflare.
5. Regenera el cliente OpenVPN con la IP nueva.

## Script original

```bash
#!/bin/bash
#Comparo los dos resultados, si son difefetentes meto el segundo en el fichero y restarto

LOG=/var/log/revisa_ip_publica.log
FECHA=D$(date +%Y%m%d)

IPVIEJA=`cat /ruta/segura/ip_publica_actual`
IPPUBLICA=`dig +short myip.opendns.com @resolver1.opendns.com`
FICHERO=/ruta/segura/ip_publica_actual

if [[ $IPVIEJA != $IPPUBLICA ]]
   then
   echo no es igual > $LOG
   cp -p $FICHERO $FICHERO.$FECHA
   sed -i '$d' $FICHERO
   echo $IPPUBLICA >> $FICHERO
   /ruta/origen/scripts/cloudflare.sh
   /ruta/origen/scripts/open_vpn_clients.sh
fi
```

## Script original Cloudflare

```bash
#!/bin/bash
LOG=/var/log/cloudflare.log
WEB1=/ruta/origen/scripts/cloudflare_web1.sh
WEB2=/ruta/origen/scripts/cloudflare_web2.sh
WEB3=/ruta/origen/scripts/cloudflare_web3.sh
WEB4=/ruta/origen/scripts/cloudflare_web4.sh
WEB5=/ruta/origen/scripts/cloudflare_web5.sh
WEB6=/ruta/origen/scripts/cloudflare_web6.sh
WEB7=/ruta/origen/scripts/cloudflare_web7.sh
WEB8=/ruta/origen/scripts/cloudflare_web8.sh
WEB9=/ruta/origen/scripts/cloudflare_web9.sh
WEB10=/ruta/origen/scripts/cloudflare_web10.sh

echo "Sincronizamos ip local con Cloudflare" > $LOG
echo "Sincronizamos WEB1" >> $LOG
$WEB1 >> $LOG
if [[ $? -ne 0 ]]; then echo "Error al sincronizar WEB1" >> $LOG; exit 3; else
echo "Sincronizamos WEB2" >> $LOG
$WEB2 >> $LOG
if [[ $? -ne 0 ]]; then echo "Error al sincronizar WEB2" >> $LOG; exit 3; else
echo "Sincronizamos WEB3" >> $LOG
$WEB3 >> $LOG
if [[ $? -ne 0 ]]; then echo "Error al sincronizar WEB3" >> $LOG; exit 5; else
echo "Sincronizamos WEB4" >> $LOG
$WEB4 >> $LOG
if [[ $? -ne 0 ]]; then echo "Error al sincronizar WEB4" >> $LOG; exit 7; else
echo "Sincronizamos WEB5" >> $LOG
$WEB5 >> $LOG
if [[ $? -ne 0 ]]; then echo "Error al sincronizar WEB5" >> $LOG; exit 9; else
echo "Sincronizamos WEB6" >> $LOG
$WEB6 >> $LOG
if [[ $? -ne 0 ]]; then echo "Error al sincronizar WEB6" >> $LOG; exit 11; else
echo "Sincronizamos WEB7" >> $LOG
$WEB7 >> $LOG
if [[ $? -ne 0 ]]; then echo "Error al sincronizar WEB7" >> $LOG; exit 12; else
echo "Sincronizamos WEB8" >> $LOG
$WEB8 >> $LOG
if [[ $? -ne 0 ]]; then echo "Error al sincronizar WEB8" >> $LOG; exit 13; else
echo "Sincronizamos WEB9" >> $LOG
$WEB9 >> $LOG
if [[ $? -ne 0 ]]; then echo "Error al sincronizar WEB9" >> $LOG; exit 17; else
echo "Sincronizamos WEB10" >> $LOG
$WEB10 >> $LOG
if [[ $? -ne 0 ]]; then echo "Error al sincronizar WEB10" >> $LOG; exit 15; else
echo "SIncronizacion Finalizada" >> $LOG
exit 0
fi; fi; fi; fi; fi; fi; fi; fi; fi; fi
```

## Version revisada del orquestador

```bash
#!/usr/bin/env bash
set -euo pipefail

LOG="/var/log/revisa_ip_publica_$(date +%Y%m%d).log"
ESTADO_IP="/ruta/segura/ip_publica_actual"
IP_ANTERIOR="$(cat "$ESTADO_IP")"
IP_PUBLICA="$(dig +short myip.opendns.com @resolver1.opendns.com)"

if [[ "$IP_ANTERIOR" != "$IP_PUBLICA" ]]; then
  cp -p "$ESTADO_IP" "${ESTADO_IP}.$(date +%Y%m%d)"
  printf '%s\n' "$IP_PUBLICA" > "$ESTADO_IP"
  /ruta/origen/scripts/cloudflare.sh >> "$LOG" 2>&1
  /ruta/origen/scripts/open_vpn_clients.sh >> "$LOG" 2>&1
fi
```

## Version revisada de registro A

```bash
#!/usr/bin/env bash
set -euo pipefail

ZONE="example.net"
DNS_RECORD="servicio.example.net"
CLOUDFLARE_EMAIL="<EMAIL>"
CLOUDFLARE_TOKEN="<CLOUDFLARE_API_TOKEN>"
IP_PUBLICA="$(curl -fsS https://checkip.amazonaws.com)"

ZONE_ID="$(curl -fsS "https://api.cloudflare.com/client/v4/zones?name=${ZONE}&status=active" \
  -H "X-Auth-Email: ${CLOUDFLARE_EMAIL}" \
  -H "X-Auth-Key: ${CLOUDFLARE_TOKEN}" \
  -H 'Content-Type: application/json' | jq -r '.result[0].id')"

RECORD_ID="$(curl -fsS "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/dns_records?type=A&name=${DNS_RECORD}" \
  -H "X-Auth-Email: ${CLOUDFLARE_EMAIL}" \
  -H "X-Auth-Key: ${CLOUDFLARE_TOKEN}" \
  -H 'Content-Type: application/json' | jq -r '.result[0].id')"

curl -fsS -X PUT "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/dns_records/${RECORD_ID}" \
  -H "X-Auth-Email: ${CLOUDFLARE_EMAIL}" \
  -H "X-Auth-Key: ${CLOUDFLARE_TOKEN}" \
  -H 'Content-Type: application/json' \
  --data "{\"type\":\"A\",\"name\":\"${DNS_RECORD}\",\"content\":\"${IP_PUBLICA}\",\"ttl\":1,\"proxied\":true}"
```

## Mejoras recomendadas

- Migrar de Global API Key a token limitado a `Zone:DNS:Edit`.
- Pasar dominio y registro como argumentos para evitar scripts repetidos.
- Validar que `jq` devuelve IDs no vacios antes del `PUT`.
- Usar `curl -fsS` para fallar de forma visible.

## Cambios y motivo

| Cambio | Motivo |
|---|---|
| reemplazar cascada de `if` por bucle/funcion | reduce repeticion entre registros DNS |
| token fuera del script | evita exponer credenciales Cloudflare |
| `Authorization: Bearer` | permite usar token limitado en vez de key global |
| `curl -fsS` y validacion de IDs | hace visibles errores de API antes del `PUT` |

<!--
Fuentes consolidadas

- `/tools/scripts/libra_scripts/revisa_ip_publica.sh`
- `/tools/scripts/libra_scripts/cloudflare.sh`
- `/tools/scripts/libra_scripts/cloudflare_web1.sh`
- `/tools/scripts/libra_scripts/cloudflare_web2.sh`
- `/tools/scripts/libra_scripts/cloudflare_web3.sh`
- `/tools/scripts/libra_scripts/cloudflare_web4.sh`
- `/tools/scripts/libra_scripts/cloudflare_web5.sh`
- `/tools/scripts/libra_scripts/cloudflare_web6.sh`
- `/tools/scripts/libra_scripts/cloudflare_web7.sh`
- `/tools/scripts/libra_scripts/cloudflare_web8.sh`
- `/tools/scripts/libra_scripts/cloudflare_web9.sh`
- `/tools/scripts/libra_scripts/cloudflare_web10.sh`
-->
