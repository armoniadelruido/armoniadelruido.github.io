---
title: Parseo rapido de logs de correo y Apache
date: 2026-07-06 00:30:00
categories: [Sistemas, Scripting]
tags: [logs, apache, postfix, dovecot, awk, grep, bash]
---

# Parseo rapido de logs de correo y Apache

Este post recopila comandos para investigar accesos, autenticaciones fallidas y origenes mas frecuentes en logs de correo y Apache.

## Top IPs en Apache

```bash
awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c | sort -nr | head
```

## Codigos HTTP mas frecuentes

```bash
awk '{print $9}' /var/log/apache2/access.log | sort | uniq -c | sort -nr
```

## Top IPs en logs SSL

```bash
awk '{print $1}' /var/log/apache2/ssl_access.log | sort | uniq -c | sort -nr | head
```

## Autenticaciones fallidas en mail

```bash
grep -i 'auth failed\|authentication failed' /var/log/mail.log
```

## Logins IMAP por usuario/IP

```bash
grep -i 'imap-login' /var/log/mail.log \
  | awk '{print $0}' \
  | sort \
  | uniq -c \
  | sort -nr
```

## Reverse DNS rapido

```bash
for ip in $(awk '{print $1}' access.log | sort -u); do
  printf '%s ' "$ip"
  dig +short -x "$ip"
done
```

## Consejos

- Acotar por fecha antes de contar.
- Separar bots conocidos de accesos reales.
- No publicar IPs completas en informes publicos.
- Guardar salidas en ficheros timestamp si la investigacion se repite.

## Jitsi y Fail2ban

```bash
awk '/http-bind\?room=/ {print $1, $4, $7}' /var/log/apache2/jitsi/access.log | sort -u
fail2ban-client status
tail -n 50 /var/log/fail2ban.log
```

## Alerta simple

```bash
curl -fsS -X POST "https://api.telegram.org/bot<TOKEN>/sendMessage" \
  -d chat_id="<CHAT_ID>" \
  --data-urlencode text="Revisar logs en <HOSTNAME>"
```

<!--
Fuentes consolidadas

- `/tools/scripts/accesos.sh`
- `/tools/scripts/submision.sh`
- `/tools/scripts/conexiones.sh`
- `/tools/scripts/conexiones_ssl.sh`
-->
