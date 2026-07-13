---
title: Oracle listener wallet export import y restore
date: 2026-07-02 00:50:00
categories: [BBDD, Oracle]
tags: [sysadmin, oracle, listener, wallet, expdp, impdp, restore]
---

# Oracle listener wallet export import y restore

Este post ayuda a revisar listeners, gestionar wallets, lanzar exports/imports, comprobar usuarios y preparar tareas basicas de recuperacion Oracle.

Parar el listener corta nuevas conexiones, aunque sesiones ya abiertas pueden seguir dependiendo de la configuracion. `impdp` puede crear o sobrescribir objetos segun parametros, asi que revisa usuario, schema y destino. Las wallets contienen material sensible: permisos restrictivos y nada de copiarlas sin control.

## Listener

```bash
lsnrctl status
lsnrctl services
lsnrctl stop
lsnrctl start
```

## Variables de entorno

```bash
export ORACLE_HOME=/opt/oracle/product/<VERSION>/dbhome_1
export ORACLE_SID=<INSTANCIA>
export PATH=$ORACLE_HOME/bin:$PATH
```

## Conexion local

```bash
sqlplus / as sysdba
```

## Estado de la instancia

```sql
select instance_name, status from v$instance;
select name, open_mode from v$database;
```

## Data Pump export

```bash
expdp <USUARIO>/<SECRETO>@<SERVICIO> \
  schemas=<ESQUEMA> \
  directory=<DIRECTORY_OBJECT> \
  dumpfile=<ESQUEMA>_%U.dmp \
  logfile=expdp_<ESQUEMA>.log
```

## Data Pump import

```bash
impdp <USUARIO>/<SECRETO>@<SERVICIO> \
  schemas=<ESQUEMA> \
  directory=<DIRECTORY_OBJECT> \
  dumpfile=<ESQUEMA>_%U.dmp \
  logfile=impdp_<ESQUEMA>.log
```

## Wallet

```bash
orapki wallet create -wallet /ruta/ejemplo/wallet -auto_login
orapki wallet add -wallet /ruta/ejemplo/wallet -trusted_cert -cert ca-example.crt
orapki wallet display -wallet /ruta/ejemplo/wallet
```

## Restore: comprobaciones previas

```bash
rman target /
```

```sql
list backup summary;
list archivelog all;
report schema;
```

## Usuarios y grants

```sql
select username, account_status from dba_users order by username;
select grantee, granted_role from dba_role_privs where grantee = '<USUARIO>';
alter user <USUARIO> account unlock;
alter user <USUARIO> identified by "<SECRETO>";
```

## Data Pump con remapeo

```bash
impdp <USUARIO>/<SECRETO>@<SERVICIO> \
  directory=<DIRECTORY_OBJECT> \
  dumpfile=<DUMPFILE> \
  logfile=impdp_<ESQUEMA>.log \
  remap_schema=<ESQUEMA_ORIGEN>:<ESQUEMA_DESTINO>
```

<!--
Fuentes consolidadas

- `Oracle.txt`
- `oracle.txt`
- `Oracle_GOL.txt`
- `Listeners_oracle.txt`
- `wallet_oracle.txt`
- `export_oracle.txt`
- `restore_oracle.txt`
- `impor_exports.txt`
- `recreacion Objetos  Oracle.txt`
- `INVESDC12.txt`
- `depurar invesdoc.txt`
- `restores11g.txt`
- `grants.txt`
- `grants_roles.txt`
- `new 26.txt`
- `new 27.txt`
- `new 38.txt`
- `new 42.txt`
- `new 44.txt`
- `new 46.txt`
- `new 48.txt`
- `new 49.txt`
- `new 58.txt`
- `new 62.txt`
- `new 70.txt`
- `new 78.txt`
-->
