---
title: Survivor DB2
date: 2025-01-16 0:31:00
categories: [BBDD, DB2]
tags: [sysdamin, bbdd, base de datos db2, tablespaces, survivor]
---

## Comando útiles de db2.

Mostrar bases de datos disponibles:
```bash
db2 list active databases
```
Mostrar configuracion de la base de datos:
```bash
db2 get db cfg for MIBBDD
```
Cambiar el archivado de logs a disco:
```bash
db2 update db cfg for MIBBDD using LOGARCHMETH1 DISK:/ruta/a/alguna/parte
```
Mostrar usuarios de una base de datos:
```bash
MySQL     : SELECT User,Host FROM mysql.user;
PostgreSQL: SELECT * FROM pg_user;
```
Mostrar las tablas de la base de datos:
```bash
db2 list tables for all
```
Mostrar las tablas de la base de datos que contengan COSA en su nombre:
```bash
db2 "select * from syscat.tables where tabname like '%COSA%' order by tabname 
```
Verificar backup completo de los logs:
```bash
db2adutl query full db BBDD
```
Verificar fecha y hora de los logs:
```bash
db2adutl query logs
```
Sacar  roles, grants, mascaras si las hay etc...de la BBDD o de una tabla dada, o tablespace:
```bash
db2look -d BBDD -z ESQUEMA -c -r -e -x -t TABLA/OBJETOABUSCAR -nofed
db2look -d BBDD -z ESQUEMA -e -m 
```
Uso del log transaccional:
```bash
db2 "select * from sysibmadm.log.utilization"
```
Ver si hay alguna transacción pendiente/en duda/enganchada:
```bash
db2 list indoubt transactions with prompting 
r 1 (Rollback, Forguet)

db2 get snapshot for application applid *LOCAL.instancia.1234556789

db2pd -transaction -db BBDD
db2pd -locks -db BBDD
db2p -transactions tran=222 -db BBDD
db2 force applications \(3333\)
```
Ver el estado de una tabla:
```bash
db2 load query table ESQUEMA.TABLA
```
Si necesita un reorg:
```bash
db2 reorg table ESQUEMA.TABLA
```
Si tiene un problema de integridad:
```bash
db2 set integrity for TABLA immediate checked
```
Si la tabla esta en load pending:
```bash
db2 "load from /RUTA/DEL/FICHERO/DONDE/HACELA/CARGA/ of del terminate into ESQUEMA.TABLA copy yes use SISTEMAARCHIVADO"
db2 "load from /RUTA/DEL/FICHERO/DONDE/HACELA/CARGA/ of del terminate into ESQUEMA.TABLA copy yes to /dev/null" <-- Si nos da igual que no se guarde en el log
```
Si esta en load pending, pero no hay ningun fichero de carga:
```bash
db2 "load from /dev/null of del terminate into ESQUEMA.TABLA copy yes use SISTEMAARCHIVADO"
db2 "load from /dev/null of del terminate into ESQUEMA.TABLA copy yes to /dev/null" <-- Si nos da igual que no se guarde en el log
```
Si la tabla está en estado backup pending:
```bash
db2 backup databse BBDD tablespace \NOMBRETABLESPACE1,NOMBRETABLESPACE2\ online use SISTEMAARCHIVADO
```

### Operativa con tablas y tablespaces.
Obtener tablas y su tamaño en kilobytes, megabytes y gigabytes (sin índices):
```bash
db2 "SELECT SUBSTR(TABSCHEMA,1,18) TABSCHEMA, SUBSTR(TABNAME,1,30) TABNAME, 
(DATA_OBJECT_P_SIZE + LONG_OBJECT_P_SIZE + LOB_OBJECT_P_SIZE + XML_OBJECT_P_SIZE) AS TOTAL_SIZE_IN_KB,
(DATA_OBJECT_P_SIZE + LONG_OBJECT_P_SIZE + LOB_OBJECT_P_SIZE + XML_OBJECT_P_SIZE)/1024 AS TOTAL_SIZE_IN_MB, 
(DATA_OBJECT_P_SIZE + LONG_OBJECT_P_SIZE + LOB_OBJECT_P_SIZE + XML_OBJECT_P_SIZE) / (1024*1024) AS TOTAL_SIZE_IN_GB 
FROM SYSIBMADM.ADMINTABINFO 
WHERE TABSCHEMA='MIESQUEMA' 
AND TABNAME IN (SELECT TABNAME FROM SYSCAT.TABLES WHERE TBSPACE='MITABLESPACE') 
ORDER BY 4"
```
Obtener tablas y su tamaño en kilobytes, megabytes y gigabytes (incluyendo índices):
```bash
db2 "SELECT SUBSTR(TABSCHEMA,1,18) TABSCHEMA, SUBSTR(TABNAME,1,30) TABNAME, 
(DATA_OBJECT_P_SIZE + INDEX_OBJECT_P_SIZE + LONG_OBJECT_P_SIZE + LOB_OBJECT_P_SIZE + XML_OBJECT_P_SIZE) AS TOTAL_SIZE_IN_KB,
(DATA_OBJECT_P_SIZE + INDEX_OBJECT_P_SIZE + LONG_OBJECT_P_SIZE + LOB_OBJECT_P_SIZE + XML_OBJECT_P_SIZE)/1024 AS TOTAL_SIZE_IN_MB, 
(DATA_OBJECT_P_SIZE + INDEX_OBJECT_P_SIZE + LONG_OBJECT_P_SIZE + LOB_OBJECT_P_SIZE + XML_OBJECT_P_SIZE) / (1024*1024) AS TOTAL_SIZE_IN_GB 
FROM SYSIBMADM.ADMINTABINFO 
WHERE TABSCHEMA='MIESQUEMA' 
AND TABNAME IN (SELECT TABNAME FROM SYSCAT.TABLES WHERE TBSPACE='MITABLESPACE') 
ORDER BY 4"
```
Ordenado por tabla y tablespaces
Obtener tamaño de las tablas en relación con su tablespace:
```bash
db2 "SELECT SUBSTR(a.TABSCHEMA,1,18) TABSCHEMA, SUBSTR(a.TABNAME,1,30) TABNAME, 
substr(b.tbspace,1,20) tbspace, 
(a.DATA_OBJECT_P_SIZE + a.LONG_OBJECT_P_SIZE + a.LOB_OBJECT_P_SIZE + a.XML_OBJECT_P_SIZE) AS TOTAL_SIZE_IN_KB,
(a.DATA_OBJECT_P_SIZE + a.LONG_OBJECT_P_SIZE + a.LOB_OBJECT_P_SIZE + a.XML_OBJECT_P_SIZE)/1024 AS TOTAL_SIZE_IN_MB, 
(a.DATA_OBJECT_P_SIZE + a.LONG_OBJECT_P_SIZE + a.LOB_OBJECT_P_SIZE + a.XML_OBJECT_P_SIZE) / (1024*1024) AS TOTAL_SIZE_IN_GB 
FROM SYSIBMADM.ADMINTABINFO a, SYSCAT.TABLES b 
WHERE a.TABSCHEMA='MIESQUEMA' AND b.TABSCHEMA='MIESQUEMA' AND a.tabname=b.tabname 
ORDER BY 5"

```
