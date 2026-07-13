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

## Original: deteccion de IP publica

Este script compara la IP publica actual con la ultima guardada. Si cambia, actualiza el fichero de estado, lanza la actualizacion DNS y despues regenera/envia el cliente OpenVPN.

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

## Original: orquestador Cloudflare

Este orquestador llama uno a uno a los scripts que actualizan cada registro DNS. Es una cadena sencilla: si falla uno, sale con codigo de error y deja rastro en el log.

```bash
#!/bin/bash
LOG=/var/log/cloudflare.log
CHAT=/ruta/origen/scripts/cloudflare_chat.sh
MEET=/ruta/origen/scripts/cloudflare_meet.sh
CLOUD=/ruta/origen/scripts/cloudflare_cloud.sh
DOCS=/ruta/origen/scripts/cloudflare_docs.sh
RAIZ=/ruta/origen/scripts/cloudflare_armoniadelruido.sh
WWW=/ruta/origen/scripts/cloudflare_www.sh
CODE=/ruta/origen/scripts/cloudflare_code.sh
RUST=/ruta/origen/scripts/cloudflare_rust.sh
JARVIS=/ruta/origen/scripts/cloudflare_jarvis.sh
VPN=/ruta/origen/scripts/cloudflare_vpn.sh

echo "Sincronizamos ip local con Cloudflare" > $LOG
echo "Sincronizamos MEET" >> $LOG
$MEET >> $LOG
if [[ $? -ne 0 ]]; then echo "Error al sincronizar MEET" >> $LOG; exit 3; else
echo "Sincronizamos CHAT" >> $LOG
$CHAT >> $LOG
if [[ $? -ne 0 ]]; then echo "Error al sincronizar CHAT" >> $LOG; exit 3; else
echo "Sincronizamos CLOUD" >> $LOG
$CLOUD >> $LOG
if [[ $? -ne 0 ]]; then echo "Error al sincronizar CLOUD" >> $LOG; exit 5; else
echo "Sincronizamos DOCS" >> $LOG
$DOCS >> $LOG
if [[ $? -ne 0 ]]; then echo "Error al sincronizar DOCS" >> $LOG; exit 7; else
echo "Sincronizamos WWW" >> $LOG
$WWW >> $LOG
if [[ $? -ne 0 ]]; then echo "Error al sincronizar WWW" >> $LOG; exit 11; else
echo "Sincronizamos CODE" >> $LOG
$CODE >> $LOG
if [[ $? -ne 0 ]]; then echo "Error al sincronizar CODE" >> $LOG; exit 12; else
echo "Sincronizamos RUST" >> $LOG
$RUST >> $LOG
if [[ $? -ne 0 ]]; then echo "Error al sincronizar RUST" >> $LOG; exit 13; else
echo "Sincronizamos VPN" >> $LOG
$VPN >> $LOG
if [[ $? -ne 0 ]]; then echo "Error al sincronizar VPN" >> $LOG; exit 15; else
echo "Sincronizo jarvis" >> $LOG
$JARVIS >> $LOG
if [[ $? -ne 0 ]]; then echo "Error al sincronizar jarvis" >> $LOG; exit 17; else
echo "SIncronizacion Finalizada" >> $LOG
exit 0
fi; fi; fi; fi; fi; fi; fi; fi; fi
```

## Original: actualizacion de un registro Cloudflare

Cada `cloudflare_*.sh` sigue el mismo patron: calcula IP publica, busca zona, busca registro y actualiza el registro A correspondiente.

```bash
#!/bin/bash

zone=example.net
dnsrecord=servicio.example.net
cloudflare_auth_email=<EMAIL>
cloudflare_auth_key=<CLOUDFLARE_API_TOKEN>

ip=$(curl -s -X GET https://checkip.amazonaws.com)

echo "Current IP is $ip"

if host $dnsrecord 1.1.1.1 | grep "has address" | grep "$ip"; then
  echo "$dnsrecord is currently set to $ip; no changes needed"
  exit
fi

zoneid=$(curl -s -X GET "https://api.cloudflare.com/client/v4/zones?name=$zone&status=active" \
  -H "X-Auth-Email: $cloudflare_auth_email" \
  -H "X-Auth-Key: $cloudflare_auth_key" \
  -H "Content-Type: application/json" | jq -r '{"result"}[] | .[0] | .id')

dnsrecordid=$(curl -s -X GET "https://api.cloudflare.com/client/v4/zones/$zoneid/dns_records?type=A&name=$dnsrecord" \
  -H "X-Auth-Email: $cloudflare_auth_email" \
  -H "X-Auth-Key: $cloudflare_auth_key" \
  -H "Content-Type: application/json" | jq -r '{"result"}[] | .[0] | .id')

curl -s -X PUT "https://api.cloudflare.com/client/v4/zones/$zoneid/dns_records/$dnsrecordid" \
  -H "X-Auth-Email: $cloudflare_auth_email" \
  -H "X-Auth-Key: $cloudflare_auth_key" \
  -H "Content-Type: application/json" \
  --data "{\"type\":\"A\",\"name\":\"$dnsrecord\",\"content\":\"$ip\",\"ttl\":1,\"proxied\":true}" | jq
```

## Original: actualizacion OpenVPN

Si la IP publica cambia, este script modifica la linea `remote` del perfil `.ovpn`, guarda copia previa y envia el fichero por correo.

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

## Original: renovacion de certificados

Este script renueva el wildcard con DNS-01 en Cloudflare, copia los PEM a una ubicacion temporal y delega la distribucion al script del servicio destino.

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

## Original: distribucion de certificados

Los scripts de distribucion copian los PEM al host destino y ejecutan alli un script de renovacion local.

```bash
#!/bin/bash
szLOG=/var/log/certis_docserver.log
FICHERO1=/etc/letsencrypt/live/example.net/privkey.pem
FICHERO2=/etc/letsencrypt/live/example.net/fullchain.pem
TEMPORAL=/tmp

scp /tmp/$FICHERO1 $FICHERO2 <USUARIO>@<HOSTNAME>:/tmp
ssh <USUARIO>@<HOSTNAME> "/ruta/origen/scripts/renova.sh"
```

## Original: envio a pfSense

Este bloque toma el certificado usado por Apache, copia certificado y clave a pfSense y ejecuta el importador PHP en el firewall.

```bash
#!/bin/bash
FECHA=$(date '+%b %d')
CERTI=`grep SSLCertificateFile /ruta/origen/apache/proxy.conf |sort -u | awk '{print $2}'`
KEY=`grep SSLCertificateKeyFile /ruta/origen/apache/proxy.conf |sort -u| awk '{print $2}'`
LIMITE=`openssl x509 -text -noout -in $CERTI |grep After|awk '{print $4, $5}'`

echo "$LIMITE"

scp -P 2222 pfsense-import-certificate.php $CERTI $KEY <USUARIO_ADMIN>@<HOSTNAME>:/ruta/destino/ssl \
  && ssh -p 2222 <USUARIO_ADMIN>@<HOSTNAME> 'php /ruta/destino/ssl/pfsense-import-certificate.php /ruta/destino/ssl/cert.pem /ruta/destino/ssl/privkey.pem'
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
