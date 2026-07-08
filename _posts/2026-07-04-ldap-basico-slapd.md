---
title: LDAP basico con slapd
date: 2026-07-04 01:20:00
categories: [Sistemas, Identidad]
tags: [sysadmin, ldap, slapd, usuarios, identidad]
---

# LDAP basico con slapd

Este post ayuda a instalar y comprobar un LDAP basico con `slapd`, crear entradas LDIF y validar busquedas.

## Instalacion

```bash
apt install -y slapd ldap-utils
dpkg-reconfigure slapd
```

## Busqueda base

```bash
ldapsearch -x -H ldap://ldap.example.local -b dc=example,dc=local
```

## Entrada LDIF de ejemplo

```ldif
dn: ou=usuarios,dc=example,dc=local
objectClass: organizationalUnit
ou: usuarios
```

Aplicar:

```bash
ldapadd -x -D cn=admin,dc=example,dc=local -W -f usuarios.ldif
```

## Comprobacion de usuario

```bash
getent passwd <USUARIO>
id <USUARIO>
```

<!--
Fuentes consolidadas

- `ldap`
-->
