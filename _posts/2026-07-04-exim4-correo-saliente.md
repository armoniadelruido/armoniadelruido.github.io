---
title: Exim4 y correo saliente
date: 2026-07-04 01:10:00
categories: [Sistemas, Correo]
tags: [sysadmin, exim4, smtp, correo, smarthost]
---

# Exim4 y correo saliente

Este post sirve para preparar envio de correo saliente desde Debian usando Exim4, smarthost SMTP y pruebas basicas de entrega.

Guarda credenciales SMTP con permisos restrictivos y revisa `/var/log/exim4/mainlog` despues de reiniciar para confirmar que no hay rechazos de autenticacion. Si envias adjuntos sensibles, como perfiles VPN, hazlo solo dentro de un entorno de confianza o cifra el fichero antes de mandarlo.

## Reconfigurar Exim4

```bash
dpkg-reconfigure exim4-config
```

Opciones habituales:

```text
mail sent by smarthost; no local mail
System mail name: example.local
SMTP smarthost: smtp.example.local::587
```

## Credenciales SMTP

Editar `/etc/exim4/passwd.client`:

```text
smtp.example.local:<USUARIO>:<PASSWORD>
```

Aplicar cambios:

```bash
update-exim4.conf
systemctl restart exim4
```

## Prueba de envio

```bash
printf 'Subject: prueba\n\nMensaje de prueba\n' | sendmail usuario@example.com
tail -f /var/log/exim4/mainlog
```

## Envio MIME con adjunto

```bash
FILE="/ruta/segura/openvpn/cliente.ovpn"
FROM="<EMAIL>"
TO="<EMAIL>"

{
  printf 'From: %s\n' "$FROM"
  printf 'To: %s\n' "$TO"
  printf 'Subject: Nueva configuracion VPN\n'
  printf 'MIME-Version: 1.0\n'
  printf 'Content-Type: multipart/mixed; boundary="FILEBOUNDARY"\n'
  printf -- '--FILEBOUNDARY\n'
  printf 'Content-Type: text/plain; charset=UTF-8\n\n'
  printf 'Adjunto perfil actualizado.\n'
  printf -- '--FILEBOUNDARY\n'
  printf 'Content-Type: application/octet-stream; name="%s"\n' "$(basename "$FILE")"
  printf 'Content-Disposition: attachment; filename="%s"\n' "$(basename "$FILE")"
  printf 'Content-Transfer-Encoding: base64\n\n'
  base64 "$FILE"
  printf -- '--FILEBOUNDARY--\n'
} | /usr/sbin/exim4 -t
```

<!--
Fuentes consolidadas

- `exim4`
-->
