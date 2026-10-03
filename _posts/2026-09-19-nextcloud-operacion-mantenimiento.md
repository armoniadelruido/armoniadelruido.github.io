---
title: "Nextcloud: operación, mantenimiento y diagnóstico"
date: 2026-09-19 10:00:00
categories: [Sistemas, Nextcloud]
tags: [sysadmin, nextcloud, occ, php, mantenimiento]
---

# Nextcloud: operación, mantenimiento y diagnóstico

Esta guía reúne las tareas recurrentes de una instalación Nextcloud: revisar el estado, activar mantenimiento, reparar la base de datos, reindexar archivos y diagnosticar PHP. Las rutas, dominios, usuarios e identificadores son marcadores anónimos.

En los ejemplos:

```text
<RUTA_NEXTCLOUD> = /var/www/nextcloud
<USUARIO_WEB>    = www-data
<USUARIO_NC>     = usuario_ejemplo
```

## Comprobación inicial

```bash
cd <RUTA_NEXTCLOUD>
sudo -u <USUARIO_WEB> php occ status
sudo -u <USUARIO_WEB> php occ config:list system
sudo -u <USUARIO_WEB> php occ check
```

La salida de `config:list system` puede contener nombres de servidor, rutas o parámetros internos. Debe revisarse antes de copiarla a un ticket o publicarla.

## Modo mantenimiento

Activar:

```bash
cd <RUTA_NEXTCLOUD>
sudo -u <USUARIO_WEB> php occ maintenance:mode --on
```

Desactivar al terminar:

```bash
sudo -u <USUARIO_WEB> php occ maintenance:mode --off
```

Si una actualización quedó interrumpida:

```bash
sudo -u <USUARIO_WEB> php occ maintenance:repair
sudo -u <USUARIO_WEB> php occ upgrade
```

Antes de reparar o actualizar se necesita una copia coherente de la configuración, la base de datos y el directorio de datos.

## Base de datos

Las advertencias del panel administrativo suelen indicar los comandos necesarios:

```bash
sudo -u <USUARIO_WEB> php occ db:add-missing-indices
sudo -u <USUARIO_WEB> php occ db:add-missing-primary-keys
sudo -u <USUARIO_WEB> php occ db:add-missing-columns
sudo -u <USUARIO_WEB> php occ db:convert-filecache-bigint
```

No es necesario ejecutar todos por rutina. Se aplican cuando el panel, la actualización o la documentación de la versión lo indique, con copia previa de la base de datos.

## Archivos que no aparecen

Para un usuario concreto:

```bash
sudo -u <USUARIO_WEB> php occ files:scan --path='<USUARIO_NC>/files'
```

Para toda la instancia:

```bash
sudo -u <USUARIO_WEB> php occ files:scan --all
```

Un escaneo global puede consumir bastante tiempo y E/S. Es mejor empezar por la cuenta o ruta afectada.

Otras comprobaciones útiles:

```bash
sudo -u <USUARIO_WEB> php occ files:cleanup
sudo -u <USUARIO_WEB> php occ integrity:check-core
sudo -u <USUARIO_WEB> php occ app:list
```

## PHP: CLI, FPM y Apache

Nextcloud puede usar una versión de PHP en la web y otra en la consola. Primero hay que identificar ambas:

```bash
php -v
update-alternatives --display php
systemctl --type=service | grep -E 'php.*fpm'
apachectl -M | grep -i php
apachectl -tD DUMP_INCLUDES | grep -i php
```

Tras cambiar la versión de FPM, se debe habilitar la configuración correspondiente y validar Apache antes de recargar:

```bash
sudo a2disconf php<VERSION_ANTERIOR>-fpm
sudo a2enconf php<VERSION_NUEVA>-fpm
sudo systemctl enable --now php<VERSION_NUEVA>-fpm
sudo apachectl configtest
sudo systemctl reload apache2
```

Después:

```bash
sudo -u <USUARIO_WEB> php occ status
sudo -u <USUARIO_WEB> php occ check
```

Los módulos de PHP deben corresponder a la versión activa. La lista exacta depende de la edición de Nextcloud y de las aplicaciones instaladas.

## Permisos

Evita aplicar permisos recursivos a ciegas. Primero revisa propietario, grupo y sistema de ficheros:

```bash
namei -l <RUTA_NEXTCLOUD>
findmnt -T <RUTA_NEXTCLOUD>
sudo -u <USUARIO_WEB> test -r <RUTA_NEXTCLOUD>/config/config.php
```

La configuración, el código y los datos tienen necesidades distintas. Los permisos deben ajustarse al método de instalación y al usuario real del servidor web.

## Cambio del directorio de datos

Nextcloud recomienda mantener la misma ruta siempre que sea posible. Cambiar `datadirectory` después de instalar requiere actualizar referencias internas y puede romper relaciones en la base de datos.

Si no existe alternativa:

1. Crear una copia restaurable de base de datos, configuración y datos.
2. Activar mantenimiento y detener los servicios web y tareas programadas.
3. Copiar los datos conservando propietarios, permisos y atributos.
4. Actualizar `datadirectory` y las referencias internas siguiendo la documentación de la versión instalada.
5. Arrancar, validar con `occ status` y probar varios usuarios antes de retirar la ruta anterior.

No se deben publicar el `config.php`, volcados SQL ni salidas completas de diagnóstico sin anonimizar. Pueden contener dominios, IP, usuarios, rutas, credenciales y secretos de instancia.

## Secuencia de diagnóstico

```bash
sudo -u <USUARIO_WEB> php <RUTA_NEXTCLOUD>/occ status
sudo -u <USUARIO_WEB> php <RUTA_NEXTCLOUD>/occ check
journalctl -u apache2 -u 'php*-fpm' --since today
tail -n 100 <RUTA_DATOS>/nextcloud.log
```

Antes de compartir la salida, sustituir cualquier dominio, dirección, nombre de usuario, ruta privada, identificador de instancia o token por un marcador semántico.

## Referencias

- [Comandos `occ` para la base de datos](https://docs.nextcloud.com/server/latest/admin_manual/occ_database.html)
- [Resolución general de problemas](https://docs.nextcloud.com/server/stable/admin_manual/issues/general_troubleshooting.html)
- [Migración a otro servidor](https://docs.nextcloud.com/server/stable/admin_manual/maintenance/migrating.html)
