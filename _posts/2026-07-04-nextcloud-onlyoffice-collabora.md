---
title: Nextcloud OnlyOffice y Collabora
date: 2026-07-04 00:10:00
categories: [Sistemas, Middleware]
tags: [sysadmin, nextcloud, onlyoffice, collabora, apache, nginx, php, ssl]
---

# Nextcloud OnlyOffice y Collabora

Este post cubre instalaciones Nextcloud/OnlyOffice, ajustes de PHP, cambio de directorio de datos, modo mantenimiento y publicacion mediante proxy web.

## Paquetes base

```bash
apt update
apt install -y apache2 mariadb-server php php-fpm php-mysql php-gd php-curl php-xml php-zip php-mbstring unzip wget
```

## Ajustes PHP habituales

```ini
memory_limit = 1024M
upload_max_filesize = 1024M
post_max_size = 1024M
max_execution_time = 360
date.timezone = Europe/Madrid
```

## Base de datos

```sql
CREATE DATABASE <BBDD> CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
CREATE USER '<USUARIO>'@'localhost' IDENTIFIED BY '<PASSWORD>';
GRANT ALL PRIVILEGES ON <BBDD>.* TO '<USUARIO>'@'localhost';
FLUSH PRIVILEGES;
```

## Apache virtualhost

```apache
<VirtualHost *:80>
    ServerName cloud.example.local
    DocumentRoot /var/www/nextcloud

    <Directory /var/www/nextcloud/>
        Require all granted
        AllowOverride All
        Options FollowSymLinks MultiViews
    </Directory>

    ErrorLog /var/log/apache2/nextcloud_error.log
    CustomLog /var/log/apache2/nextcloud_access.log combined
</VirtualHost>
```

## Mover directorio de datos

```bash
sudo -u www-data php /var/www/nextcloud/occ maintenance:mode --on
mkdir -p /srv/nextcloud/data
rsync -a /var/www/nextcloud/data/ /srv/nextcloud/data/
chown -R www-data:www-data /srv/nextcloud/data
```

Actualizar `config.php`:

```php
'datadirectory' => '/srv/nextcloud/data',
```

Actualizar tabla de storages si aplica:

```sql
use <BBDD>;
update oc_storages set id='local::/srv/nextcloud/data/' where id='local::/var/www/nextcloud/data/';
```

```bash
sudo -u www-data php /var/www/nextcloud/occ maintenance:mode --off
```

## Modo solo lectura de datos

```bash
find /srv/nextcloud/data -type f -print0 | xargs -0 chmod 0440
find /srv/nextcloud/data -type d -print0 | xargs -0 chmod 0550
chmod 0640 /srv/nextcloud/data/.ocdata
```

## OnlyOffice con Docker

```bash
docker run -d \
  --name onlyoffice \
  --restart=always \
  -p 443:443 \
  -e JWT_SECRET=<SECRETO> \
  -v /srv/onlyoffice/logs:/var/log/onlyoffice \
  -v /srv/onlyoffice/data:/var/www/onlyoffice/Data \
  onlyoffice/documentserver
```

## Obtener secreto configurado

```bash
docker exec <CONTAINER_ID> /var/www/onlyoffice/documentserver/npm/json \
  -f /etc/onlyoffice/documentserver/local.json \
  'services.CoAuthoring.secret.session.string'
```

## Collabora reverse proxy

```apache
AllowEncodedSlashes NoDecode
SSLProxyEngine On
ProxyPreserveHost On

ProxyPass        /browser https://<IP_INTERNA>:9980/browser retry=0
ProxyPassReverse /browser https://<IP_INTERNA>:9980/browser

ProxyPass        /hosting/discovery https://<IP_INTERNA>:9980/hosting/discovery retry=0
ProxyPassReverse /hosting/discovery https://<IP_INTERNA>:9980/hosting/discovery

ProxyPassMatch   "/cool/(.*)/ws$" wss://<IP_INTERNA>:9980/cool/$1/ws nocanon
ProxyPass        /cool https://<IP_INTERNA>:9980/cool
ProxyPassReverse /cool https://<IP_INTERNA>:9980/cool
```

## Sincronizacion externa de ficheros

Si se sincronizan ficheros hacia un almacenamiento usado por Nextcloud, conviene evitar escrituras directas sin mantenimiento y reindexado.

```bash
sudo -u www-data php /var/www/nextcloud/occ maintenance:mode --on
rsync -avh /origen/ /srv/nextcloud/data/<USUARIO>/files/
chown -R www-data:www-data /srv/nextcloud/data
sudo -u www-data php /var/www/nextcloud/occ files:scan --all
sudo -u www-data php /var/www/nextcloud/occ maintenance:mode --off
```

Para pruebas, usar antes:

```bash
rsync -avhn /origen/ /srv/nextcloud/data/<USUARIO>/files/
```

## Script relacionado

`sync_files_a_nextcloud.sh` monta un origen remoto de Nextcloud, sincroniza rutas origen locales hacia el destino montado y lanza una correccion remota de permisos.

<!--
Fuentes consolidadas

- `nexcloud_php8_debian11_install`
- `nextcloud_con_ssl`
- `nextcloud/cambio_data_files`
- `nextcloud/config.php`
- `nextcloud/nexcloud_15_proxmox_apache`
- `nextcloud/nextclooud_proxmox`
- `nextcloud/php`
- `nextcloud/read_only_mode`
- `nextcloud/resolucion-de-problemas-nextcloud-1.pdf`
- `onlyoffice/generate.json`
- `onlyoffice/local.json`
- `onlyoffice/problemas`
- `/tools/scripts/sync_files_a_nextcloud.sh`
- `/tools/scripts/sync_docs_a_nextcloud.sh._v1`
- `/tools/scripts/sync_docs_a_cirro.sh`
-->
