---
title: Backup homelab Nextcloud runbook
date: 2026-07-10 00:10:00
categories: [Sistemas, Backups]
tags: [nextcloud, backup, restore, rsync, sql]
---

# Backup homelab Nextcloud runbook

Este runbook junta las piezas de backup de Nextcloud: base de datos, directorio de datos, configuracion del sistema, movimiento mensual y retencion.

## Uso en la infraestructura

Este runbook protege la continuidad del servicio Nextcloud. Cubre la BBDD, el directorio de datos, la aplicacion y las copias que salen hacia almacenamiento externo.

| Rol | Servicio | Impacto si falla |
|---|---|---|
| Backup diario | Nextcloud | No hay punto reciente para recuperar datos o BBDD |
| Copia externa | Almacenamiento de backups | Una perdida local tambien puede afectar a las copias |
| Retencion | Backups antiguos | El destino puede llenarse o borrar mas de lo debido |
| Restore | Nextcloud | No se puede validar recuperacion tras corrupcion o migracion |

Depende de que los mounts remotos funcionen, de credenciales SQL guardadas en ruta segura y de que el modo mantenimiento se active/desactive correctamente.

## Objetivo

- Tener copia diaria de SQL y datos.
- Separar backup local rapido de copia mensual externa.
- Poder restaurar BBDD y filesystem en orden controlado.
- Evitar borrar origenes antes de confirmar destino.

## Secuencia recomendada

```text
1. Activar mantenimiento en Nextcloud.
2. Generar dump SQL.
3. Copiar config/app/data con rsync.
4. Desactivar mantenimiento.
5. Empaquetar copias cerradas.
6. Mover a destino externo.
7. Aplicar retencion.
8. Probar restauracion periodicamente.
```

## Patron recomendado: backup SQL

```bash
MYSQL_CNF="/ruta/segura/mysql.cnf"
DB_NAME="<BBDD_NEXTCLOUD>"
DESTINO_SQL="/ruta/destino/backups/sql"

sudo -u www-data php /ruta/origen/nextcloud/occ maintenance:mode --on
trap 'sudo -u www-data php /ruta/origen/nextcloud/occ maintenance:mode --off' EXIT

mysqldump --defaults-extra-file="$MYSQL_CNF" "$DB_NAME" \
  > "$DESTINO_SQL/nextcloud_$(date +%Y%m%d).sql"
```

## Patron recomendado: backup de datos

```bash
rsync -aHAX --numeric-ids /ruta/origen/nextcloud-data/ /ruta/destino/nextcloud-data/
rsync -aHAX --numeric-ids /ruta/origen/nextcloud-app/ /ruta/destino/nextcloud-app/
rsync -aHAX --numeric-ids /ruta/origen/etc/ /ruta/destino/etc/
```

## Patron recomendado: movimiento mensual

```bash
FECHA="$(date +%Y%m%d)"
tar czf "/tmp/nextcloud-data_${FECHA}.tar.gz" -C /ruta/destino nextcloud-data
mount <HOSTNAME>:/ruta/destino/backups-mensuales /mnt/backups
trap 'umount -l /mnt/backups 2>/dev/null || true' EXIT
mv "/tmp/nextcloud-data_${FECHA}.tar.gz" /mnt/backups/nextcloud/
```

## Patron recomendado: retencion local SQL

`dumpea.sh` genera dumps diarios y `muevo_sqls.sh` los empaqueta semanalmente. Si el destino externo no monta, el host no debe llenar ni `/bck_local` ni `/tmp` mientras espera al siguiente intento.

```bash
RETENER_DUMPS=15
RETENER_TARS=2

timeout 60s mount <HOSTNAME>:/ruta/destino/backups-mensuales /mnt/backups

find /bck_local/sqls -maxdepth 1 -type f -name 'bck_*' -printf '%T@ %p\n' \
  | sort -rn \
  | awk -v keep="$RETENER_DUMPS" 'NR>keep { $1=""; sub(/^ /, ""); print }' \
  | while IFS= read -r fichero; do rm -f -- "$fichero"; done

find /tmp -maxdepth 1 -type f -name 'sqls.*.tar.gz' -printf '%T@ %p\n' \
  | sort -rn \
  | awk -v keep="$RETENER_TARS" 'NR>keep { $1=""; sub(/^ /, ""); print }' \
  | while IFS= read -r fichero; do rm -f -- "$fichero"; done
```

Con esta retencion quedan los ultimos 15 dumps diarios sueltos y los ultimos 2 tar SQL. En la practica da varias semanas de margen operativo aunque el envio externo falle temporalmente.

## Patron recomendado: retencion

```bash
find /ruta/destino/backups/sql -type f -name '*.sql*' -mtime +90 -print -delete
find /ruta/destino/backups/nextcloud -type f -name '*.tar.gz' -mtime +90 -print -delete
```

## Restore minimo

1. Parar servicios web o activar mantenimiento.
2. Restaurar filesystem con propietarios originales.
3. Importar SQL en una BBDD limpia.
4. Revisar `config.php` y rutas `datadirectory`.
5. Ejecutar `occ maintenance:repair`.
6. Ejecutar `occ files:scan --all` si aplica.
7. Desactivar mantenimiento y validar login/subida/descarga.

## Checklist

- El dump SQL no esta vacio.
- El destino esta montado donde toca.
- El backup no depende solo de `/tmp`.
- El mount remoto tiene timeout y no bloquea el cron indefinidamente.
- `/bck_local/sqls` conserva solo los ultimos 15 dumps diarios.
- `/tmp` conserva solo los ultimos 2 tar SQL.
- Hay logs con fecha.
- La retencion usa `-type f` y patrones concretos.
- Hay al menos una prueba de restore documentada.

## Scripts originales relacionados

| Script | Uso |
|---|---|
| `dumpea.sh` | dump SQL Nextcloud |
| `backup_filesystem.sh` | copia local de filesystem y datos |
| `muevo_sqls.sh` | movimiento de SQLs a almacenamiento externo |
| `muevo_filesystems_neptuno.sh` | empaquetado mensual de filesystems |
| `depuro_neptuno_90.sh` | retencion en destino externo |
| `backup_nube_neptuno.sh` | copia mensual del arbol de datos |
| `sincro_nubes.sh` | replica entre instancias/hosts |

<!--
Fuentes consolidadas

- `/tools/scripts/nextcloud_scripts/backup_filesystem.sh`
- `/tools/scripts/nextcloud_scripts/dumpea.sh`
- `/tools/scripts/nextcloud_scripts/muevo_sqls.sh`
- `/tools/scripts/nextcloud_scripts/muevo_filesystems_neptuno.sh`
- `/tools/scripts/nextcloud_scripts/depuro_neptuno_90.sh`
- `/tools/scripts/nextcloud_scripts/backup_nube_neptuno.sh`
- `/tools/scripts/nextcloud_scripts/sincro_nubes.sh`
- `/tools/scripts/nextcloud_scripts/import_sql.sh`
-->
