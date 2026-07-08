---
title: Sincronizar ficheros con SFTP y rsync en homelab
date: 2026-07-06 00:50:00
categories: [Storage & backups, Backups]
tags: [sftp, rsync, backup, homelab, cron]
---

# Sincronizar ficheros con SFTP y rsync en homelab

Este post documenta un flujo de sincronizacion con SFTP y `rsync`: descarga, subida, borrado controlado, historico y logs.

## Flujo recomendado

1. Descargar ficheros remotos a directorio temporal.
2. Validar que la descarga contiene lo esperado.
3. Subir/copiar al destino final.
4. Mover los ficheros procesados a historico.
5. Borrar en origen solo si todo lo anterior ha ido bien.

## Directorios

```bash
IN="/srv/sftp/in"
OUT="/srv/sftp/out"
HIST="/srv/sftp/hist/$(date +%Y%m%d_%H%M%S)"
LOG="/var/log/sftp-rsync.log"
```

## SFTP batch

```text
cd /remoto/out
mget *
rm *
bye
```

Ejecutar:

```bash
sftp -b batch.sftp <USUARIO>@<HOSTNAME>
```

## Rsync a destino

```bash
rsync -avh --remove-source-files "$OUT/" <USUARIO>@<HOSTNAME>:/destino/
```

## Historico

```bash
mkdir -p "$HIST"
find "$OUT" -type f -print0 | xargs -0 -I{} mv {} "$HIST/"
```

## Log sencillo

```bash
printf '[%s] Comienza SFTP\n' "$(date --iso-8601=seconds)" >> "$LOG"
printf '[%s] Finaliza descarga\n' "$(date --iso-8601=seconds)" >> "$LOG"
printf '[%s] Finaliza limpieza remota\n' "$(date --iso-8601=seconds)" >> "$LOG"
```

## Precauciones

- Evitar `rm .*` y patrones ambiguos.
- Usar directorios temporales por ejecucion.
- No borrar remoto si falla la copia al destino.
- Registrar fecha, host, numero de ficheros y codigo de salida.

<!--
Fuentes consolidadas

- `/tools/scripts/sftp_provas/prova.sh`
- `/tools/scripts/sftp_provas/log/SFTP.log`
- `/tools/scripts/sftp_provas/log/SFTP.logecho`
-->
