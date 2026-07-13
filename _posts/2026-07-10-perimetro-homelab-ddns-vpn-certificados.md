---
title: Perimetro homelab DDNS VPN certificados
date: 2026-07-10 00:20:00
categories: [Sistemas, Redes]
tags: [cloudflare, openvpn, certbot, pfsense, ddns]
---

# Perimetro homelab DDNS VPN certificados

El perimetro del homelab depende de tres automatismos coordinados: actualizar DNS cuando cambia la IP publica, regenerar perfiles VPN si usan IP directa y renovar certificados para servicios publicados.

## Uso en la infraestructura

Este post cubre la capa que mantiene accesibles los servicios publicados desde una conexion con IP publica cambiante.

| Rol | Servicio afectado | Impacto si falla |
|---|---|---|
| DDNS | Cloudflare | Los FQDN apuntan a una IP antigua |
| VPN | OpenVPN | Los clientes no conectan desde fuera |
| TLS | Nextcloud, Docs, Jitsi, pfSense | Certificados caducados o no desplegados |
| Frontal | pfSense/HAProxy | Servicios publicados con certificado incorrecto |

La prioridad operativa es que VPN y DNS funcionen primero; despues se valida TLS y frontales.

## Mapa de dependencias

```text
IP publica cambia
├── actualizar registros Cloudflare
├── revisar perfiles OpenVPN
└── mantener certificados y frontales independientes de la IP

Certificado wildcard renovado
├── distribuir PEM a servicios internos
├── importar en pfSense
├── actualizar HAProxy
└── reiniciar servicios consumidores
```

## Patron recomendado: DDNS Cloudflare

```bash
IP_PUBLICA="$(curl -fsS https://checkip.amazonaws.com)"
curl -fsS -X PUT "https://api.cloudflare.com/client/v4/zones/<ZONE_ID>/dns_records/<RECORD_ID>" \
  -H "Authorization: Bearer <CLOUDFLARE_API_TOKEN>" \
  -H 'Content-Type: application/json' \
  --data "{\"type\":\"A\",\"name\":\"<FQDN>\",\"content\":\"${IP_PUBLICA}\",\"ttl\":1,\"proxied\":true}"
```

## Patron recomendado: VPN

Si el cliente OpenVPN apunta a un FQDN, basta con DDNS. Si apunta a IP directa, hay que regenerar o editar el `.ovpn`:

```bash
sed -i.bak "/^remote /c\remote <FQDN> 1194" /ruta/segura/openvpn/cliente.ovpn
```

## Patron recomendado: certificados

```bash
certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials /ruta/segura/cloudflare.ini \
  --non-interactive \
  -d example.net \
  -d '*.example.net'
```

## Patron recomendado: pfSense y HAProxy

El flujo seguro es copiar certificado y clave a una ruta temporal controlada, importarlos con script CLI, asignarlos a la GUI y actualizar frontends HAProxy que apuntaban al certificado anterior.

```bash
scp -P 2222 /ruta/destino/certs/fullchain.pem /ruta/destino/certs/privkey.pem \
  <USUARIO_ADMIN>@<HOSTNAME>:/ruta/destino/ssl/
ssh -p 2222 <USUARIO_ADMIN>@<HOSTNAME> \
  'php /ruta/destino/ssl/pfsense-import-certificate.php /ruta/destino/ssl/fullchain.pem /ruta/destino/ssl/privkey.pem'
```

## Checklist

- Tokens Cloudflare con permisos minimos.
- Credenciales certbot en `/ruta/segura/...` con permisos `600`.
- OpenVPN preferiblemente con FQDN en lugar de IP.
- Certificados probados antes de reiniciar servicios.
- Fecha de expiracion registrada tras desplegar.

## Scripts originales relacionados

| Script | Uso |
|---|---|
| `revisa_ip_publica.sh` | detectar cambio de IP publica |
| `cloudflare.sh` y `cloudflare_*.sh` | actualizar registros DNS |
| `open_vpn_clients.sh` | actualizar y enviar perfil OpenVPN |
| `renueva_certis.sh` | renovar wildcard Let's Encrypt |
| `pfsense_certs.sh` | llevar certificados a pfSense |
| `certis_nextcloud.sh`, `certis_docserver.sh`, `certis_jitsiking.sh` | distribuir PEMs a servicios |

<!--
Fuentes consolidadas

- `/tools/scripts/libra_scripts/revisa_ip_publica.sh`
- `/tools/scripts/libra_scripts/cloudflare.sh`
- `/tools/scripts/libra_scripts/cloudflare_chat.sh`
- `/tools/scripts/libra_scripts/cloudflare_cloud.sh`
- `/tools/scripts/libra_scripts/cloudflare_docs.sh`
- `/tools/scripts/libra_scripts/cloudflare_meet.sh`
- `/tools/scripts/libra_scripts/cloudflare_www.sh`
- `/tools/scripts/libra_scripts/cloudflare_code.sh`
- `/tools/scripts/libra_scripts/cloudflare_rust.sh`
- `/tools/scripts/libra_scripts/cloudflare_jarvis.sh`
- `/tools/scripts/libra_scripts/cloudflare_vpn.sh`
- `/tools/scripts/libra_scripts/cloudflare_armoniadelruido.sh`
- `/tools/scripts/libra_scripts/open_vpn_clients.sh`
- `/tools/scripts/libra_scripts/envia_cliente_openvpn.sh`
- `/tools/scripts/libra_scripts/renueva_certis.sh`
- `/tools/scripts/libra_scripts/pfsense_certs.sh`
- `/tools/scripts/libra_scripts/piholes_certs.sh`
- `/tools/scripts/libra_scripts/certis_nextcloud.sh`
- `/tools/scripts/libra_scripts/certis_docserver.sh`
- `/tools/scripts/libra_scripts/certis_jitsiking.sh`
- `/tools/scripts/libra_scripts/pfsense-import-certificate.php`
-->
