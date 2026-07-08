---
title: Rsync crontab y backups caseros
date: 2026-07-04 00:20:00
categories: [Storage & backups, Backups]
tags: [sysadmin, rsync, crontab, backup, linux, homelab]
---

# Rsync crontab y backups caseros

Este post sirve para preparar copias simples con `rsync`, programarlas con cron y documentar circuitos basicos de backup en homelab.

## Rsync local

```bash
rsync -avh --delete /origen/ /destino/
```

## Rsync remoto por SSH

```bash
rsync -avh --delete -e ssh /origen/ <USUARIO>@<HOSTNAME>:/destino/
```

## Excluir rutas

```bash
rsync -avh --delete \
  --exclude 'cache/' \
  --exclude '*.tmp' \
  /origen/ /destino/
```

## Script base

```bash
#!/usr/bin/env bash
set -euo pipefail

ORIGEN="/origen/"
DESTINO="/destino/"
LOG="/var/log/backup-rsync.log"

rsync -avh --delete "$ORIGEN" "$DESTINO" >> "$LOG" 2>&1
```

## Cron

```cron
30 2 * * * /usr/local/sbin/backup-rsync.sh
```

## Comandos utiles de crontab

```bash
crontab -l
crontab -e
crontab -u <USUARIO> -l
```

<!--
Fuentes consolidadas

- `circuito de backups`
- `crontabs_orion`
- `crontabs_orion_segundas_copias.txt`
- `rsync`
- `rsync_samples`
- `deb9/crontabs`
-->
