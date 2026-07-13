---
title: Backups de configuracion de servicios
date: 2026-07-10 00:50:00
categories: [Sistemas, Backups]
tags: [backup, configuracion, nginx, pfsense, proxmox]
---

# Backups de configuracion de servicios

Los datos importan, pero en un homelab la recuperacion depende tambien de configuraciones: `/etc`, nginx, certificados, pfSense, Proxmox y scripts propios.

## Uso en la infraestructura

Este post protege la parte que suele ser reconstruible pero lenta: configuraciones, certificados, reverse proxies, firewalls, hipervisores y scripts de operacion.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| Config sistema | `/etc` | Reinstalar un host requiere rehacer ajustes a mano |
| Reverse proxy | nginx/Apache | Servicios publicados no levantan igual tras restore |
| Firewall | pfSense | Se pierde configuracion de reglas, NAT, VPN o HAProxy |
| Virtualizacion | Proxmox | Se complica reconstruir jobs, storage y configuracion |
| Scripts | `/tools` | Se pierden automatizaciones operativas |

Es complementario al backup de datos: sin configuracion, los datos pueden existir pero el servicio tarda mucho mas en volver.

## Que copiar

- `/ruta/origen/etc`: configuracion base del sistema.
- `/ruta/origen/tools`: scripts y utilidades propias.
- `/ruta/origen/nginx`: virtualhosts y reverse proxies.
- `/ruta/origen/letsencrypt`: certificados y renovaciones.
- Export XML de pfSense.
- Configuracion de Proxmox y jobs de backup.

## Patron recomendado: rsync de configuracion

```bash
rsync -aHAX --numeric-ids /ruta/origen/etc/ /ruta/destino/config/etc/
rsync -aHAX --numeric-ids /ruta/origen/tools/ /ruta/destino/config/tools/
rsync -aHAX --numeric-ids <HOSTNAME>:/ruta/origen/nginx/ /ruta/destino/config/nginx/
```

## Patron recomendado: empaquetado mensual

```bash
FECHA="$(date +%Y%m%d)"
tar czf "/ruta/destino/config/etc_${FECHA}.tar.gz" -C /ruta/destino/config etc
tar czf "/ruta/destino/config/tools_${FECHA}.tar.gz" -C /ruta/destino/config tools
```

## Patron recomendado: pfSense

Este ejemplo presupone que `cookies.txt` contiene una sesion valida. En pfSense normalmente hay que hacer login antes, capturar el token CSRF y comprobar que la descarga resultante es un XML real, no una pagina de login. Despues de descargar, valida que el fichero no esta vacio y que empieza como configuracion pfSense antes de dar el backup por bueno.

```bash
curl -fsS -b cookies.txt -o "/ruta/destino/pfsense/pfsense_$(date +%Y%m%d).xml" \
  'https://<HOSTNAME>/diag_backup.php?download=download'
```

## Restore de configuracion

1. Restaurar primero en ruta temporal.
2. Comparar con `diff -ruN`.
3. Aplicar cambios de forma selectiva.
4. Validar sintaxis del servicio.
5. Recargar, no reiniciar, si el servicio lo permite.

## Comprobaciones

```bash
nginx -t
apachectl configtest
systemctl status servicio
tar tzf /ruta/destino/config/etc_YYYYMMDD.tar.gz >/dev/null
```

## Scripts originales relacionados

| Script | Uso |
|---|---|
| `me_traigo_la_config_de_libra.sh` | recuperar configs nginx y letsencrypt |
| `sync_files_a_nextcloud.sh` | sincronizar `/tools` hacia Nextcloud |
| `backup_filesystem.sh` | copia de `/etc`, `/tools` y datos |
| `proxmox_backup_collar.sh` | backup de configuracion/rutas a PBS |
| `pfsense_backups_auto.sh` | export XML de pfSense |

<!--
Fuentes consolidadas

- `/tools/scripts/libra_scripts/me_traigo_la_config_de_libra.sh`
- `/tools/scripts/libra_scripts/sync_files_a_nextcloud.sh`
- `/tools/scripts/nextcloud_scripts/backup_filesystem.sh`
- `/tools/scripts/proxmox_backup_collar.sh`
- `/tools/scripts/proxmox-pve-config-backup.sh`
- `/tools/scripts/pfsense/pfsense_backups_auto.sh`
-->
