---
title: Sincronizar Nextcloud con Infomaniak kDrive usando rclone
date: 2026-07-11 00:20:00
categories: [Sistemas, Backups]
tags: [nextcloud, infomaniak, kdrive, rclone, webdav, backup]
---

# Sincronizar Nextcloud con Infomaniak kDrive usando rclone

En este procedimiento montaremos una copia externa de datos de Nextcloud hacia Infomaniak kDrive usando `rclone` sobre WebDAV.

La idea es sencilla: configurar un remote `kdrive:` en `rclone`, probar acceso, y lanzar una copia controlada desde el host Nextcloud hacia una carpeta remota.

## Uso En La Infraestructura

Lo usaremos como copia externa adicional. No sustituye al backup local ni al dump SQL, pero añade una capa fuera del host y del almacenamiento local.

| Rol | Pieza | Impacto si falla |
|---|---|---|
| Copia externa | Infomaniak kDrive | No hay copia fuera del entorno local |
| Sincronización | `rclone` WebDAV | Los datos no se actualizan en destino |
| Consistencia | modo mantenimiento Nextcloud | Puede copiarse mientras hay cambios en curso |
| Seguridad | app password | Credenciales expuestas permiten acceso al kDrive |

## Requisitos

- Cuenta Infomaniak con kDrive activo.
- URL WebDAV del kDrive.
- App password específica para `rclone`.
- `rclone` instalado en el host Nextcloud.
- Acceso al directorio de datos de Nextcloud.
- Permisos para ejecutar `occ` como usuario web.
- Ruta de logs escribible.

## Instalar rclone

En Debian o Ubuntu:

```bash
sudo apt update
sudo apt install -y rclone
```

Comprobar versión:

```bash
rclone version
```

## Configurar el Remote kDrive

Infomaniak kDrive puede configurarse como WebDAV compatible con Nextcloud.

Opción directa:

```bash
rclone config create kdrive webdav \
  url https://<ID_KDRIVE>.connect.kdrive.infomaniak.com/remote.php/webdav \
  vendor nextcloud \
  user "<EMAIL>" \
  pass "<APP_PASSWORD>"
```

Opción recomendada para no dejar la contraseña en el historial:

```bash
rclone config
```

Selecciona:

- tipo: `webdav`
- vendor: `nextcloud`
- URL: `https://<ID_KDRIVE>.connect.kdrive.infomaniak.com/remote.php/webdav`
- usuario: `<EMAIL>`
- contraseña: app password de Infomaniak

Protege el fichero de configuración:

```bash
chmod 600 ~/.config/rclone/rclone.conf
```

Si el script se ejecutará como `root`, revisa también:

```bash
/root/.config/rclone/rclone.conf
```

## Probar la conexión

Listar carpetas:

```bash
rclone lsd kdrive:
```

Listar contenido:

```bash
rclone lsf kdrive:
```

Crear y borrar una carpeta de prueba:

```bash
rclone mkdir kdrive:prueba_rclone
rclone lsd kdrive:
rclone purge kdrive:prueba_rclone
```

## Script 

Usaremos `rclone copy`. Copia ficheros nuevos o modificados, pero no borra en kDrive lo que ya exista allí y no esté en origen.

```bash
#!/usr/bin/env bash
set -Eeuo pipefail
IFS=$'\n\t'

NC_DATA="/ruta/origen/nextcloud/data"
NC_PATH="/ruta/origen/nextcloud"
NC_USER="www-data"
KREMOTE="kdrive:"
KDST_DIR="nextcloud_backup"
LOG="/var/log/nextcloud_to_kdrive.log"

COMMON_OPTS=(
  --checkers=16
  --transfers=8
  --checksum
  --fast-list
  --tpslimit 8
  --retries 5
  --retries-sleep 10s
  --log-file="$LOG"
  --log-level INFO
)

EXCLUDES=(
  --exclude "/appdata_*/**"
  --exclude "/*/files_trashbin/**"
  --exclude "/*/files_versions/**"
  --exclude "/*/cache/**"
  --exclude "/*/uploads/**"
  --exclude "/*/thumbnails/**"
  --exclude "/*/files_*/**/previews/**"
  --exclude "/updater-*/**"
  --exclude "/ownbackup/**"
)

logline() {
  printf '[%s] %s\n' "$(date '+%F %T')" "$*" | tee -a "$LOG"
}

enable_maintenance() {
  logline "Habilitando mantenimiento"
  sudo -u "$NC_USER" php "$NC_PATH/occ" maintenance:mode --on
}

disable_maintenance() {
  logline "Deshabilitando mantenimiento"
  sudo -u "$NC_USER" php "$NC_PATH/occ" maintenance:mode --off || true
}

trap disable_maintenance EXIT

logline "===== INICIO SYNC Nextcloud -> kDrive ====="
enable_maintenance

rclone mkdir "${KREMOTE}${KDST_DIR}" || true
rclone copy "$NC_DATA" "${KREMOTE}${KDST_DIR}" "${COMMON_OPTS[@]}" "${EXCLUDES[@]}"

logline "===== FIN SYNC ====="
```

## Por Qué Usar `copy`

`rclone copy` es el modo más prudente para empezar.

Ventajas:

- No borra en destino.
- Reduce riesgo si el origen queda incompleto por error.
- Permite usar kDrive como copia acumulativa.

Inconvenientes:

- Si borramos mucho contenido en origen, seguirá ocupando espacio en kDrive.
- Requiere limpieza manual o política de retención.

## Variante Con Espejo Exacto

Si quieremos que kDrive sea un espejo exacto del origen, usaremos `rclone sync`.

Importante: `sync` borra en destino lo que no exista en origen. Lanzar siempre antes con `--dry-run`.

```bash
rclone sync "$NC_DATA" "${KREMOTE}${KDST_DIR}" \
  "${COMMON_OPTS[@]}" "${EXCLUDES[@]}" \
  --delete-excluded \
  --dry-run
```

Si el resultado es correcto, quitamos `--dry-run`.

## Versionado Con `--backup-dir`

Una forma más segura de usar `sync` es guardar antes los ficheros borrados o sobrescritos.

```bash
DATE="$(date +%F)"
BACKUP_DIR="${KREMOTE}${KDST_DIR}_versions/${DATE}"

rclone mkdir "${KREMOTE}${KDST_DIR}" || true
rclone mkdir "${KREMOTE}${KDST_DIR}_versions" || true
rclone mkdir "$BACKUP_DIR" || true

rclone sync "$NC_DATA" "${KREMOTE}${KDST_DIR}" \
  "${COMMON_OPTS[@]}" "${EXCLUDES[@]}" \
  --backup-dir "$BACKUP_DIR"
```

Con este patrón:

- `kdrive:nextcloud_backup` contiene el estado actual.
- `kdrive:nextcloud_backup_versions/YYYY-MM-DD` contiene ficheros reemplazados o borrados.

## Limpiar Versiones Antiguas

Si activamos versionado, conviene purgar carpetas antiguas cuando ya no las necesites.

```bash
cleanup_old_versions() {
  limit_ts="$(date -d '90 days ago' +%s)"

  rclone lsf "${KREMOTE}${KDST_DIR}_versions" --dirs-only --format "p" --fast-list \
  | while read -r folder; do
      folder="${folder%/}"
      if date -d "$folder" +%s >/dev/null 2>&1; then
        folder_ts="$(date -d "$folder" +%s)"
        if [ "$folder_ts" -lt "$limit_ts" ]; then
          rclone purge "${KREMOTE}${KDST_DIR}_versions/${folder}"
        fi
      fi
    done
}
```

Activaremos esta limpieza solo después de validar que las carpetas de versión tienen formato de fecha esperado.

## Exclusiones

Para evitar subir datos regenerables o demasiado voluminosos:

- `appdata_*`
- `files_trashbin`
- `files_versions`
- `cache`
- `uploads`
- `thumbnails`
- `previews`
- `updater-*`
- `ownbackup`

Si queremos conservar papelera o versiones de Nextcloud en el backup externo, no excluyas esas rutas.

## Programar La Ejecución

Cron genérico:

```cron
30 3 * * * /ruta/origen/scripts/nextcloud_to_kdrive.sh
```

Para evitar solapes:

```bash
flock -n /var/lock/nextcloud_to_kdrive.lock /ruta/origen/scripts/nextcloud_to_kdrive.sh
```

Evita ejecutarlo a la vez que:

- dumps SQL;
- backup local;
- movimiento a NAS;
- tareas pesadas de mantenimiento.

## Comprobaciones Posteriores

Revisar log:

```bash
tail -100 /var/log/nextcloud_to_kdrive.log
```

Comprobar destino:

```bash
rclone lsd kdrive:
rclone lsf kdrive:nextcloud_backup --max-depth 1
```

Asegurar que Nextcloud no queda en mantenimiento:

```bash
sudo -u www-data php /ruta/origen/nextcloud/occ maintenance:mode --off
```

## Troubleshooting

| Problema | Revisión |
|---|---|
| Error de autenticación | regenerar app password y revisar usuario |
| Funciona manualmente pero no desde cron | comprobar usuario, `PATH` y `rclone.conf` |
| No encuentra el remote | ejecutar `rclone config file` |
| WebDAV va lento | bajar `--transfers`, `--checkers` o ajustar `--tpslimit` |
| Nextcloud queda en mantenimiento | ejecutar `occ maintenance:mode --off` |
| Se suben demasiados ficheros | revisar exclusiones y probar con `--dry-run` |

## Checklist

- `rclone` instalado.
- Remote `kdrive:` probado.
- App password no escrita en scripts ni notas.
- `rclone.conf` con permisos `600`.
- `rclone lsd kdrive:` funciona.
- Script probado con `--dry-run`.
- Decidido si usar `copy` o `sync`.
- Si se usa `sync`, probado primero con `--dry-run`.
- Si se usa versionado, revisada la limpieza antes de activar `purge`.
- Logs revisados tras la primera ejecución.

<!--
Fuentes consolidadas

- /home/alexaid/Documentos/wikis/infomaniak/sincronizacion-nextcloud-kdrive-infomaniak.txt
- /tools/scripts/nextcloud_scripts/kdrive/LEEME
- /tools/scripts/nextcloud_scripts/kdrive/nextcloud_to_kdrive.sh
- /tools/scripts/nextcloud_scripts/kdrive/scripts_mejorados/nextcloud_to_kdrive.sh
-->
