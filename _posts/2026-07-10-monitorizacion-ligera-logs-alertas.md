---
title: Monitorizacion ligera logs y alertas
date: 2026-07-10 00:30:00
categories: [Sistemas, Monitorizacion]
tags: [logs, apache, fail2ban, telegram, jitsi]
---

# Monitorizacion ligera logs y alertas

Una monitorizacion ligera puede cubrir bastante con `tail`, `awk`, `sort`, Fail2ban y alertas puntuales por Telegram.

## Uso en la infraestructura

Este post cubre vigilancia basica sin desplegar un stack pesado. Sirve para detectar actividad rara en servicios publicados y recibir avisos simples.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| Revision web | Apache/HTTP | Errores 4xx/5xx o abuso pasan desapercibidos |
| Salas | Jitsi | No hay visibilidad rapida de uso |
| Bloqueos | Fail2ban | Ataques o falsos positivos no se detectan |
| Avisos | Telegram | Los scripts no notifican incidencias |

No sustituye a una monitorizacion completa; es la primera linea para hosts pequenos o servicios caseros.

## Patron recomendado: Apache IPs y codigos

```bash
awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c | sort -nr | head
awk '{print $9}' /var/log/apache2/access.log | sort | uniq -c | sort -nr
```

## Patron recomendado: Jitsi salas detectadas

```bash
awk '/http-bind\?room=/ {print $1, $4, $7}' /var/log/apache2/jitsi/access.log \
  | awk -F? '{print $1, $2}' \
  | grep -v '<IP_INTERNA>' \
  | sort -u > /var/log/jitsi_rooms.log
```

## Patron recomendado: Fail2ban

```bash
fail2ban-client status
fail2ban-client status sshd
tail -n 50 /var/log/fail2ban.log
```

## Patron recomendado: Telegram

```bash
TOKEN="<TOKEN>"
CHAT_ID="<CHAT_ID>"
curl -fsS -X POST "https://api.telegram.org/bot${TOKEN}/sendMessage" \
  -d chat_id="$CHAT_ID" \
  --data-urlencode text="Alerta: revisar logs en <HOSTNAME>"
```

## Rutina diaria

1. Revisar top IPs y 4xx/5xx.
2. Revisar Fail2ban y jails activas.
3. Revisar logs de servicios publicados.
4. Enviar alerta solo si hay cambio o umbral superado.
5. Guardar evidencia con fecha si se investiga un incidente.

## Mejoras

- Pasar de comandos manuales a scripts idempotentes.
- Evitar `cat | grep`; usar `awk` o `grep` directamente.
- Centralizar logs por servicio.
- No enviar tokens por linea de comandos compartida si queda historial.

## Scripts originales relacionados

| Script | Uso |
|---|---|
| `accesos.sh`, `conexiones.sh`, `conexiones_ssl.sh` | parseos rapidos de logs |
| `jitsi.sh`, `jitsi1.sh` | extraer salas desde logs Jitsi |
| `libra_bot.sh`, `libra_bot2.sh` | enviar resumen a Telegram |
| `f2ban_status.sh` | informe rapido Fail2ban |
| `unban.sh` | desbaneo manual |

<!--
Fuentes consolidadas

- `/tools/scripts/accesos.sh`
- `/tools/scripts/conexiones.sh`
- `/tools/scripts/conexiones_ssl.sh`
- `/tools/scripts/submision.sh`
- `/tools/scripts/libra_scripts/jitsi.sh`
- `/tools/scripts/libra_scripts/jitsi1.sh`
- `/tools/scripts/libra_scripts/jitsi_live.sh`
- `/tools/scripts/libra_scripts/libra_bot.sh`
- `/tools/scripts/libra_scripts/libra_bot2.sh`
- `/tools/scripts/libra_scripts/f2ban_status.sh`
- `/tools/scripts/libra_scripts/unban.sh`
-->
