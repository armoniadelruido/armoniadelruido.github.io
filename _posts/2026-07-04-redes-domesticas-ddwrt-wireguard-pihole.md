---
title: Redes domesticas DD-WRT WireGuard y Pi-hole
date: 2026-07-04 00:50:00
categories: [Sistemas, Redes]
tags: [redes, dd-wrt, wireguard, vpn, pihole, router, wireshark]
---

# Redes domesticas DD-WRT WireGuard y Pi-hole

Este post ayuda a documentar tareas de red domestica: repetidores, VPN WireGuard, resolucion DNS con Pi-hole y pruebas basicas de conectividad.

## WireGuard: servidor

```ini
[Interface]
Address = 10.10.10.1/24
ListenPort = 51820
PrivateKey = <PRIVATE_KEY>

[Peer]
PublicKey = <PUBLIC_KEY_CLIENTE>
AllowedIPs = 10.10.10.2/32
```

## WireGuard: cliente

```ini
[Interface]
Address = 10.10.10.2/24
PrivateKey = <PRIVATE_KEY_CLIENTE>
DNS = <IP_DNS>

[Peer]
PublicKey = <PUBLIC_KEY_SERVIDOR>
Endpoint = vpn.example.local:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

## Comprobaciones

```bash
wg show
ip route
ping <IP_INTERNA>
dig example.local @<IP_DNS>
```

## Wireshark basico

Filtros utiles:

```text
ip.addr == <IP_INTERNA>
tcp.port == 443
dns
icmp
```

## Consultar DNS en Cloudflare por API

```bash
curl -s -X GET \
  "https://api.cloudflare.com/client/v4/zones/<ZONE_ID>/dns_records?type=A&name=<FQDN>" \
  -H "X-Auth-Email: usuario@example.com" \
  -H "X-Auth-Key: <TOKEN>" \
  -H "Content-Type: application/json" \
  | jq -r '.result[0].content'
```

<!--
Fuentes consolidadas

- `ddwr-repetidor`
- `DHCPs DLINK.ods`
- `Router:WRT150N.pdf`
- `soluciondevpnbasadaenraspberrypi-160521192915.pdf`
- `VPN_SAURON`
- `wireguard.txt`
- `wireshark-basico.odt`
- `/tools/scripts/chuleta_api_ips`
-->
