---
title: Exim4 y correo saliente
date: 2026-07-04 01:10:00
categories: [Sistemas, Correo]
tags: [sysadmin, exim4, smtp, correo, smarthost]
---

# Exim4 y correo saliente

Este post sirve para preparar envio de correo saliente desde Debian usando Exim4, smarthost SMTP y pruebas basicas de entrega.

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

<!--
Fuentes consolidadas

- `exim4`
-->
