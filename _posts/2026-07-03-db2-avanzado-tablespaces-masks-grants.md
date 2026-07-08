---
title: DB2 avanzado tablespaces masks grants y movimientos
date: 2026-07-03 00:20:00
categories: [BBDD, DB2]
tags: [sysadmin, db2, tablespaces, grants, masks, export, import]
---

# DB2 avanzado tablespaces masks grants y movimientos

Este post cubre tareas DB2 de segundo nivel: revision de tablespaces, grants, mascaras, movimientos de tablas, export/import y runstats.

## Tablespaces

```bash
db2 list tablespaces show detail
db2 "select tbsp_name, tbsp_state, tbsp_used_pages, tbsp_free_pages from sysibmadm.tbsp_utilization"
```

## Crear tablespace

```bash
db2 "create regular tablespace <TABLESPACE> pagesize 32k managed by automatic storage extentsize 32 prefetchsize automatic bufferpool <BUFFERPOOL>"
```

## Mover tabla

```bash
db2 "call sysproc.admin_move_table('<ESQUEMA>', '<TABLA>', '<TABLESPACE_DATOS>', '<TABLESPACE_INDICES>', '<TABLESPACE_LOBS>', '', '', '', '', '', 'MOVE')"
```

## Export e import IXF

```bash
db2 "export to /ruta/ejemplo/<TABLA>.ixf of ixf select * from <ESQUEMA>.<TABLA>"
db2 "import from /ruta/ejemplo/<TABLA>.ixf of ixf insert into <ESQUEMA>.<TABLA>"
```

## Grants

```bash
db2 "grant select on table <ESQUEMA>.<TABLA> to user <USUARIO>"
db2 "grant select, insert, update, delete on table <ESQUEMA>.<TABLA> to group <GRUPO>"
```

## Masks y permisos

```bash
db2 "select tabschema, tabname, secpolicyname from syscat.tables where secpolicyname is not null"
db2 "select tabschema, tabname, colname, maskname from syscat.columnmasks"
```

## Runstats

```bash
db2 "runstats on table <ESQUEMA>.<TABLA> with distribution and detailed indexes all"
```

<!--
Fuentes consolidadas

- `db2_comands.txt`
- `EAHPC001_movimiento_tablas.txt`
- `mover_tablas.txt`
- `pagesize_tbs.txt`
- `Mascaras.txt`
- `grants.txt`
- `grants_roles.txt`
- `nomenclautra_tablas_tablespaces.txt`
- `tabla_unavailable.txt`
- `new 4 (2).txt`
- `new 12.txt`
- `new 16.txt`
- `new 17.txt`
- `new 21.txt`
- `new 35.txt`
- `new 36.txt`
- `new 47.txt`
- `new 51.txt`
- `new 54.txt`
- `new 69.txt`
- `new 83.txt`
- `new 86.txt`
- `new 87.txt`
- `new 89.txt`
-->
