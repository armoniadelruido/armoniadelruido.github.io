---
title: Certificados openssl
date: 2024-11-19 0:31:00
categories: [Sistemas, Certificados]
tags: [Certificados, ssl, https, apache, openssl]
---
# Crear un certificado Multidominio, o también SAN.

Algunas veces tenemos que crear un certificado para varios dominios, o simplemente para poner otro CN en el certificado. 
Los navegadores marcaran como inseguro un site, si un certificado no tiene Subject Alternative Name (SAN).

## Claves privadas y certificados

Al quitar la passphrase de una clave privada, el servicio arranca de forma mas comoda, pero la clave queda mas sensible. Guarda estos ficheros con permisos restrictivos, normalmente `600`, propietario correcto y nunca en repositorios. Los ejemplos son perfectos para laboratorio o certificados internos; para servicios publicados conviene usar una CA reconocida o el procedimiento corporativo.


## Vamos al lío.

Generamos par de llaves.
```bash
openssl genrsa -des3 -out servidor.key 4096
```

Ponemos passphrase y que no la pida mas.
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
Y esto sería todo, ya tendríamos un certificado listo para usar con dos CNs.
