---
title: Salesforce integracion y proxy
date: 2026-07-03 01:20:00
categories: [Sistemas, Middleware]
tags: [sysadmin, salesforce, oauth, api, proxy, apache]
---

# Salesforce integracion y proxy

Este post sirve para preparar pruebas de autenticacion OAuth, validar llamadas API y documentar configuraciones de proxy hacia Salesforce.

## OAuth: placeholders

```bash
curl -X POST https://login.salesforce.com/services/oauth2/token \
  -d 'grant_type=client_credentials' \
  -d 'client_id=<CLIENT_ID>' \
  -d 'client_secret=<SECRETO>'
```

## Consulta API

```bash
curl -H "Authorization: Bearer <TOKEN>" \
  https://<INSTANCIA>.my.salesforce.com/services/data/vXX.X/
```

## Proxy Apache

```apache
ProxyPreserveHost On
ProxyPass /salesforce/ https://<INSTANCIA>.my.salesforce.com/
ProxyPassReverse /salesforce/ https://<INSTANCIA>.my.salesforce.com/
RequestHeader set X-Forwarded-Proto "https"
```

<!--
Fuentes consolidadas

- `salesforce.txt`
-->
