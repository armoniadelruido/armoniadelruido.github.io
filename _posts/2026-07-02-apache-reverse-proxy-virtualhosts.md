---
title: Apache reverse proxy y virtualhosts
date: 2026-07-02 00:10:00
categories: [Sistemas, Middleware]
tags: [sysadmin, apache, httpd, reverse-proxy, virtualhost, ssl]
---

# Apache reverse proxy y virtualhosts

Este post cubre la creacion de virtualhosts, redirecciones HTTPS y publicacion de aplicaciones mediante reverse proxy con Apache/httpd.

## Instalacion basica

```bash
yum install -y httpd mod_ssl
systemctl enable httpd
systemctl start httpd
systemctl status httpd
```

En Debian/Ubuntu:

```bash
apt install -y apache2 libapache2-mod-proxy-html
a2enmod proxy proxy_http proxy_wstunnel ssl headers rewrite
systemctl enable apache2
systemctl restart apache2
```

## Virtualhost simple

```apache
<VirtualHost *:80>
    ServerName app.example.local
    DocumentRoot /var/www/app

    ErrorLog /var/log/httpd/app-error.log
    CustomLog /var/log/httpd/app-access.log combined

    <Directory /var/www/app>
        Require all granted
        Options -Indexes +FollowSymLinks
        AllowOverride None
    </Directory>
</VirtualHost>
```

## Reverse proxy HTTP

```apache
<VirtualHost *:443>
    ServerName app.example.local

    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/app.example.local.crt
    SSLCertificateKeyFile /etc/pki/tls/private/app.example.local.key

    ProxyPreserveHost On
    ProxyPass / http://backend.example.local:8080/
    ProxyPassReverse / http://backend.example.local:8080/

    RequestHeader set X-Forwarded-Proto "https"
    RequestHeader set X-Forwarded-Port "443"

    ErrorLog /var/log/httpd/app-proxy-error.log
    CustomLog /var/log/httpd/app-proxy-access.log combined
</VirtualHost>
```

## Redireccion de HTTP a HTTPS

```apache
<VirtualHost *:80>
    ServerName app.example.local
    Redirect permanent / https://app.example.local/
</VirtualHost>
```

## Comprobaciones rapidas

```bash
apachectl configtest
httpd -M | grep -E 'proxy|ssl|headers|rewrite'
curl -Ik https://app.example.local/
tail -f /var/log/httpd/app-proxy-error.log
```

## Proxy con backend externo

```apache
ProxyPreserveHost On
ProxyPass /api/ https://api.example.local/
ProxyPassReverse /api/ https://api.example.local/
RequestHeader set X-Forwarded-Proto "https"
```

<!--
Fuentes consolidadas

- `apacheinstall.txt`
- `README_apacheinstaler.txt`
- `Apache_sec.txt`
- `httpd_proxy_sample.txt`
- `httpd_proxy_sample2.txt`
- `Reverse_Proxy.txt`
- `wildcard.txt`
- `GI_apache_PRE.txt`
- `salesforce.txt`
- `new 7.txt`
- `new 57.txt`
- `new 64.txt`
-->
