---
title: Survivor SQL
date: 2025-01-16 0:31:00
categories: [BBDD]
tags: [sysdamin, sql, MariaDB, Mysql, postgress]
---

## Comando útiles SQL.

Mostrar usuarios de una base de datos:
```bash
MySQL     : SELECT User,Host FROM mysql.user;
PostgreSQL: SELECT * FROM pg_user;
```
Mostrar bases de datos disponibles:
```bash
MySQL     : SHOW DATABASES;
PostgreSQL: \l
```
Mostrar las tablas de la base de datos nextcloud:
```bash
MySQL     : USE nextcloud; SHOW TABLES;
PostgreSQL: \c nextcloud; \d
```
Quit Database:
```bash
MySQL     : quit
PostgreSQL: \q
```
Info sacada de aquí:
```bash
https://docs.nextcloud.com/server/15/admin_manual/configuration_database/linux_database_configuration.html
```

## Resetear password de root:
A quien no le ha pasado, vamos a ello:

systemctl stop mariadb
mysqld_safe --skip-grant-tables & 
mysqld_safe --skip-grant-tables --skip-networking &( si nos queremos asegurar que no pueda entrar nadie)
mysql -u root

FLUSH PRIVILEGES;

Encontre esta info, no recuerdo de donde la saqué:
```bash
For MySQL 5.7.6 and newer as well as MariaDB 10.1.20 and newer, use the following command:
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';

For MySQL 5.7.5 and older as well as MariaDB 10.1.20 and older, use:
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('new_password');
```

A mi, me ha funcionado este:
```bash
UPDATE mysql.user SET authentication_string = PASSWORD('password') WHERE User = 'root' AND Host = 'localhost';
FLUSH PRIVILEGES;
```

```bash
MariaDB [(none)]> USE mysql;
MariaDB [(none)]> UPDATE user SET password=PASSWORD('YourNewPasswordHere') WHERE User='root' AND Host = 'localhost';
MariaDB [(none)]> FLUSH PRIVILEGES;
```

Ahora matar el proceso mysqld_safe --skip-grant-tables
```bash
kill -9
systemctl start mariadb
```
Y listos, ya tendriamos la pass cambiada.

Ahora damos acceso desde cualquier parte:
```bash
GRANT ALL PRIVILEGES ON bbdd.* TO 'usuariobbdd'@'%';
```
Damos acceso solo a un usuaro, y solo desde una ip:
```bash
GRANT ALL PRIVILEGES ON *.* TO 'USUARIO'@'IPHOST' IDENTIFIED BY 'PASSWORD' WITH GRANT OPTION;
```
Operativa con usuarios y bbdd:
```bash
CREATE USER 'USUARIO'@'%' IDENTIFIED BY 'PASSWORD';
grant select BBDD.* to 'USUARIO'@'%';
GRANT SELECT ON BBDD.* TO 'USUARIO'@'%';
FLUSH PRIVILEGES;

REVOKE ALL PRIVILEGES ON *.* FROM 'USUARIO'@'%';

UPDATE 

DROP USER 'USUARIO'@'localhost';
DROP USER 'USUARIO'@'%';
delete FROM mysql.user where User = 'USUARIO' and Host != '%';


CREATE USER 'usuario'@'localhost' IDENTIFIED VIA mysql_native_password;
SET PASSWORD FOR 'usuario'@'localhost' = PASSWORD('password');
CREATE DATABASE IF NOT EXISTS `database`;
GRANT ALL PRIVILEGES ON `database`.* TO 'usuario'@'localhost';
```


Si da problemas al importar una bbdd por tema de codificacion (utf8 etc etc)

```bash
show variables like '%collation%';
show variables like '%char%';


MariaDB [(none)]> show variables like '%collation%';
+----------------------+-------------------+
| Variable_name        | Value             |
+----------------------+-------------------+
| collation_connection | utf8_general_ci   |
| collation_database   | latin1_swedish_ci |
| collation_server     | latin1_swedish_ci |
+----------------------+-------------------+
3 rows in set (0.00 sec)

MariaDB [(none)]> show variables like '%char%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)



mysql> show variables like '%collation%';
+----------------------+-------------------+
| Variable_name        | Value             |
+----------------------+-------------------+
| collation_connection | latin1_swedish_ci |
| collation_database   | latin1_swedish_ci |
| collation_server     | latin1_swedish_ci |
+----
------------------+-------------------+
3 rows in set (0.00 sec)

mysql> show variables like '%char%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | latin1                     |
| character_set_connection | latin1                     |
| character_set_database   | latin1                     |
| character_set_results    | latin1                     |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
7 rows in set (0.00 sec)

SHOW VARIABLES WHERE Variable_name LIKE 'character\_set\_%' OR Variable_name LIKE 'collation%';

```
```bash
SELECT User,Host FROM mysql.user, where User=bp;

update mysql.user
set Host = 
```

Si quiero importar todas las BBDD de un sql :
```bash
mysqldump -uroot -p --all-databases > dumpeo.sql
```
Si solo una:
```bash
mysqldump -uroot -p database > dumpeo.sql
```
Si no se cuales hay, voy a verlas:
```bash
mysql -uroot -p
show databases;
```

```bash
DROP DATABASE nombre bbdd
```
