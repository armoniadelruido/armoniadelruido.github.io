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

## Patron de log para SFTP

```text
YYYY-MM-DD HH:MM:SS - Comienza SFTP
YYYY-MM-DD HH:MM:SS - Finaliza descarga
YYYY-MM-DD HH:MM:SS - Se eliminan archivos temporales
YYYY-MM-DD HH:MM:SS - Finaliza limpieza remota
```

Este patron permite comprobar rapidamente si la descarga, el borrado local y la limpieza remota han terminado correctamente.

## Patron rsync con lista de rutas

```bash
#!/usr/bin/env bash
set -euo pipefail

DESTINO="<USUARIO>@<HOSTNAME>:/destino/"
LOG="/var/log/rsync-homelab.log"

rutas=(
  "/home/<USUARIO>/Documentos"
  "/home/<USUARIO>/Imagenes"
  "/opt/tools"
)

for ruta in "${rutas[@]}"; do
  printf '[%s] Sincronizando %s\n' "$(date --iso-8601=seconds)" "$ruta" >> "$LOG"
  rsync -avh --delete "$ruta/" "$DESTINO$(basename "$ruta")/" >> "$LOG" 2>&1
done
```

## Local vs SSH

Destino local:

```bash
rsync -avh --delete /origen/ /mnt/backup/origen/
```

Destino remoto:

```bash
rsync -avh --delete -e ssh /origen/ <USUARIO>@<HOSTNAME>:/backup/origen/
```

## Script relacionado

`copio_cosas_collar_local.sh` aplica este patron a varias rutas origen locales y destinos locales de backup.

## Scripts Nextcloud relacionados

- `backup_filesystem.sh`: usa `rsync` para copiar rutas origen hacia destino local/remoto.
- `muevo_sqls.sh`: empaqueta SQLs y los mueve a destino remoto.
- `muevo_filesystems_neptuno.sh`: empaqueta filesystems y los mueve a destino mensual.
- `backup_nube_neptuno.sh`: sincroniza el arbol de Nextcloud hacia destino mensual.
- `muevo_tools_etc_urano.sh`: empaqueta `etc` y `tools` y los mueve a destino externo.

## Matriz origen destino

| Caso | Ejemplo | Precaucion |
|---|---|---|
| Local a local | `/ruta/origen/` a `/ruta/destino/` | comprobar espacio libre |
| Local a NFS | `/ruta/origen/` a `/mnt/destino/` | validar mount antes de copiar |
| SSH remoto | `<HOSTNAME>:/ruta/origen/` a `/ruta/destino/` | usar clave dedicada |
| Empaquetado | `tar.gz` mensual | no borrar origen sin confirmar copia |

Checklist minimo antes de automatizar:

- Probar con `rsync -n`.
- Registrar salida en `/var/log/<SCRIPT>.log`.
- Usar `trap` si hay mounts temporales.
- Usar patrones concretos en limpiezas con `find`.

<!--
Fuentes consolidadas

- `circuito de backups`
- `crontabs_orion`
- `crontabs_orion_segundas_copias.txt`
- `rsync`
- `rsync_samples`
- `deb9/crontabs`
- `/tools/scripts/sftp_provas/log/SFTP.log`
- `/tools/scripts/sftp_provas/log/SFTP.logecho`
- `/tools/scripts/copio_collar_a_urano.sh`
- `/tools/scripts/copio_cosas_a_urano.sh`
- `/tools/scripts/copio_FSs_a_urano.sh`
- `/tools/scripts/muevo_cosas_a_urano.sh`
- `/tools/scripts/copio_cosas_a_saturno.sh`
- `/tools/scripts/muevo_cosas_a_saturno.sh`
- `/tools/scripts/copio_cosas_collar_local.sh`
- `/tools/scripts/copio_tools_al_collar_local.sh`
- `/tools/scripts/sync_files_a_nextcloud.sh`
- `/tools/scripts/sync_docs_a_nextcloud.sh._v1`
- `/tools/scripts/sync_docs_a_cirro.sh`
-->
