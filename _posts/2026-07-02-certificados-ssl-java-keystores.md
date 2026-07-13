---
title: Certificados SSL y Java Keystores
date: 2026-07-02 00:30:00
categories: [Sistemas, Certificados]
tags: [sysadmin, Certificados, ssl, openssl, jks, keytool, jarsigner]
---

# Certificados SSL y Java Keystores

Este post sirve para generar CSR, revisar certificados, convertir formatos, gestionar keystores Java y validar firmas de JAR.

## Crear clave y CSR con OpenSSL

```bash
openssl genrsa -out app.example.local.key 4096
openssl req -new -sha256 -key app.example.local.key -out app.example.local.csr
```

## Certificado autofirmado con SAN

```bash
openssl req -x509 -nodes -newkey rsa:4096 \
  -keyout app.example.local.key \
  -out app.example.local.crt \
  -days 365 \
  -subj "/CN=app.example.local" \
  -addext "subjectAltName=DNS:app.example.local,DNS:alias.example.local"
```

## Ver contenido de certificado

```bash
openssl x509 -in app.example.local.crt -noout -text
openssl x509 -in app.example.local.crt -noout -dates
openssl x509 -in app.example.local.crt -noout -issuer -subject
```

## Comprobar certificado remoto

```bash
openssl s_client -connect app.example.local:443 -servername app.example.local </dev/null
```

## Convertir PEM a PKCS12

```bash
openssl pkcs12 -export \
  -in app.example.local.crt \
  -inkey app.example.local.key \
  -out app.example.local.p12 \
  -name app
```

## Importar en JKS

```bash
keytool -importkeystore \
  -srckeystore app.example.local.p12 \
  -srcstoretype PKCS12 \
  -destkeystore app.jks \
  -deststoretype JKS
```

## Listar JKS

```bash
keytool -list -v -keystore app.jks
```

## Importar CA en truststore

```bash
keytool -importcert \
  -alias ca-example \
  -file ca-example.crt \
  -keystore truststore.jks
```

## Firmar JAR

```bash
jarsigner -keystore app.jks aplicacion.jar app
jarsigner -verify -verbose -certs aplicacion.jar
```

## Renovacion de certificado

```bash
openssl x509 -in app.example.local.crt -noout -dates
keytool -list -v -keystore app.jks | grep -E 'Alias name|Valid from'
systemctl reload httpd
```

## Certbot con Apache

```bash
apt install -y certbot python3-certbot-apache
certbot --apache -d app.example.local -m usuario@example.com --agree-tos
certbot renew --dry-run
```

## Scripts relacionados

`libra_scripts` documenta renovacion wildcard con `certbot --dns-cloudflare`, copia controlada de PEMs e importacion de certificados en pfSense/HAProxy.

## DNS-01 con Cloudflare

```bash
certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials /ruta/segura/cloudflare.ini \
  --non-interactive \
  -d example.net \
  -d '*.example.net'
```

El fichero de credenciales debe tener permisos restrictivos:

```bash
chmod 600 /ruta/segura/cloudflare.ini
```

<!--
Fuentes consolidadas

- `openssl.txt`
- `SSLs.txt`
- `jks.txt`
- `javachuleta.txt`
- `java certis.txt`
- `jarsigner.txt`
- `KeystoreTomcat.txt`
- `ciphers_Tomcats.txt`
- `iway_proc_certificats.txt`
- `control_certis_nagios.txt`
- `new 34.txt`
- `new 57.txt`
- `creacion_certis_ssl`
-->
