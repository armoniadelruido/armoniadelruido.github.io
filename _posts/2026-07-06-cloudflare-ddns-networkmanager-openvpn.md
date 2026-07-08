---
title: Cloudflare DDNS y NetworkManager OpenVPN
date: 2026-07-06 00:20:00
categories: [Sistemas, Redes]
tags: [cloudflare, ddns, dns, vpn, networkmanager, bash]
---

# Cloudflare DDNS y NetworkManager OpenVPN

Este post cubre un patron para consultar registros DNS en Cloudflare, comparar la IP publica actual y actualizar perfiles OpenVPN gestionados por NetworkManager.

## Variables

```bash
CLOUDFLARE_API_TOKEN="<CLOUDFLARE_API_TOKEN>"
ZONE_ID="<ZONE_ID>"
FQDN="vpn.example.net"
NM_CONNECTION="<VPN_CONNECTION>"
```

## IP publica actual

```bash
ip_actual="$(curl -fsS https://api.ipify.org)"
```

## IP publicada en Cloudflare

```bash
ip_dns="$(curl -fsS \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records?type=A&name=$FQDN" \
  | jq -r '.result[0].content')"
```

## Actualizar perfil OpenVPN

```bash
nmcli connection modify "$NM_CONNECTION" +vpn.data "remote=$ip_dns"
nmcli connection reload
```

## Patron para varios perfiles

```bash
for conexion in "<VPN_1>" "<VPN_2>" "<VPN_3>"; do
  nmcli connection modify "$conexion" +vpn.data "remote=$ip_dns"
done
nmcli connection reload
```

## Modo dry-run

```bash
printf 'IP publica: %s\nIP DNS: %s\n' "$ip_actual" "$ip_dns"
printf 'Actualizaria %s con remote=%s\n' "$NM_CONNECTION" "$ip_dns"
```

## Recomendaciones

- Usar tokens Cloudflare limitados a la zona DNS necesaria.
- No usar `X-Auth-Key` global si se puede usar `Authorization: Bearer`.
- Guardar tokens en variables de entorno o fichero `.env` con permisos `600`.
- Añadir logs y salida diferenciada para cambios/no cambios.

<!--
Fuentes consolidadas

- `/tools/scripts/saco_ip_cloudflare.sh`
- `/tools/scripts/saco_ip_cloudflare.sh.ORIG`
- `/tools/scripts/saco_ip_carraca_cloudflare.sh`
- `/tools/scripts/chuleta_api_ips`
-->
