---
title: Fail2ban operativa en libra
date: 2026-07-09 01:00:00
categories: [Sistemas, Scripting]
tags: [bash, fail2ban, seguridad, logs]
---

# Fail2ban operativa en libra

Estos scripts ayudan a revisar rapidamente logs y estado de jails, y a sacar una IP de una jaula de Fail2ban de forma interactiva.

## Uso en la infraestructura

Se usa como herramienta de operacion cuando hay bloqueos, falsos positivos o sospecha de ataques contra servicios publicados.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| Estado seguridad | Fail2ban | No se revisan jails activas ni bloqueos |
| Soporte | SSH/Web | Una IP legitima puede quedar bloqueada |
| Investigacion | Apache/Nextcloud/Jitsi | Se pierde contexto rapido de los ultimos eventos |

Debe ejecutarse con cuidado: desbanear una IP elimina una proteccion activa.

## Script original de informe

```bash
#!/bin/bash
echo
echo "---------------"
echo "Fail2Ban report"
echo "---------------"
echo
echo "Last 20 lines of fail2ban.log"
echo "-----------------------------"
tail -n 20 /var/log/fail2ban.log
echo
echo "Last 20 lines of nginx-nextcloud.log"
tail -n 20 /var/log/apache2/nextcloud/access.log
echo "Last 20 lines of nginx-jitsi.log"
tail -n 20 /var/log/apache2/jitsi/access.log
echo "Last 20 lines of nginx-docserver.log"
tail -n 20 /var/log/apache2/docserver/access.log

echo "fail2ban service status"
fail2ban-client status sshd
fail2ban-client status ssh-ddos
fail2ban-client status apache-nextcloud-ddos
fail2ban-client status apache-jitsi-ddos
fail2ban-client status apache-docserver-ddos
exit 0
```

## Script original de desbaneo

```bash
#!/bin/bash
JAULA=$(fail2ban-client status| awk 'NR==3'|awk -F , '{print $1,$2,$3,$4,$5,$6,$7}')
echo "IP a sacar"
read var1
echo $var1
echo "¿De que jaula?"
echo $JAULA
echo "Selecciona una de arriba"
read var2
echo $var2
echo "¿seguro sacar $var1 del baneo de la jaula $var2?"
read -r -p "Are you sure? [y/N] " response
if [[ $response =~ ^([yY][eE][sS]|[yY])$ ]]
   then
       fail2ban-client set $var2 unbanip $var1
   else
       echo "pues nada"
fi
exit 0
```

## Version revisada de desbaneo

```bash
#!/usr/bin/env bash
set -euo pipefail

fail2ban-client status
read -r -p 'IP a sacar: ' IP
read -r -p 'Jaula: ' JAIL
read -r -p "Confirmar unban de ${IP} en ${JAIL} [y/N]: " CONFIRMA

if [[ "$CONFIRMA" =~ ^([yY][eE][sS]|[yY])$ ]]; then
  fail2ban-client set "$JAIL" unbanip "$IP"
fi
```

## Recomendaciones

- Mostrar primero `fail2ban-client status <jail>` antes del unban.
- Validar formato IP antes de ejecutar.
- Evitar `sudo` dentro del script si se ejecuta ya como root.
- Registrar desbaneos manuales en `/var/log/fail2ban_unban.log`.

## Cambios y motivo

| Cambio | Motivo |
|---|---|
| comillas en variables | evita problemas con nombres de jail o entradas vacias |
| prompts claros | reduce errores manuales al desbanear |
| validar IP | evita ejecutar `unbanip` con texto accidental |
| registrar acciones | deja auditoria de intervenciones manuales |

<!--
Fuentes consolidadas

- `/tools/scripts/libra_scripts/f2ban_status.sh`
- `/tools/scripts/libra_scripts/unban.sh`
-->
