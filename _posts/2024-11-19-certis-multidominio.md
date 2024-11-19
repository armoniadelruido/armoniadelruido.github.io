---
title: Certificados Multidominio
date: 2024-11-19 17:31:00
categories: [Inicio]
tags: [Certificados, ssl]
---
# Crear un certificado Multidominio

Algunas veces tenemos que crear un certificado para varios dominios, esta es la manera en la que lo hago:

## Subtítulo

Más contenido aquí...

### Código

Genero llave
```bash
openssl genrsa -des3 -out servidor.key 4096
```

pongo passphrase y hago que no la pida mas
```bash
mv servidor.key servidor.key.old
openssl rsa -in servidor.key.old -out servidor.key
```

Listo

```bash
openssl req -new -sha256 -key servidor.key -out servidor.csr
openssl x509 -req -days 3650 -in server.csr -signkey server.key -out server.crt
openssl req -out sslcert.csr -newkey rsa:4096 -nodes -keyout private.key -config san.cnf
```

chuleta multidominio

```bash
cp /etc/ssl/openssl.cnf /tmp
echo '[ subject_alt_name ]' >> /tmp/openssl.cnf
echo 'subjectAltName = DNS:www.example.com, DNS:site1.example.com, DNS:site2.example.com' >> /tmp/openssl.cnf
openssl req -x509 -nodes -newkey rsa:2048 \
  -config /tmp/openssl.cnf \
  -extensions subject_alt_name \
  -keyout www.example.com.key \
  -out www.example.com.pem \
  -subj '/C=XX/ST=XXXX/L=XXXX/O=XXXX/OU=XXXX/CN=www.example.com/emailAddress=postmaster@example.com'
```

chuleta copy-paste

```bash
cp /etc/ssl/openssl.cnf /tmp
echo '[ subject_alt_name ]' >> /tmp/openssl.cnf
echo 'subjectAltName = DNS:alegalu.sytes.com, DNS:devnullteamcloud.sytes.com' >> /tmp/openssl.cnf
openssl genrsa -des3 -out multidom.key rsa:4096 \
  -config /tmp/openssl.cnf \
  -extensions subject_alt_name \
  -keyout multidom.key \
  -out multidom.pem \
  -subj '/C=AD/ST=XXXX/L=XXXX/O=XXXX/OU=XXXX/CN=alegalu.sytes.com/emailAddress=postmaster@example.com'
  ```
  
