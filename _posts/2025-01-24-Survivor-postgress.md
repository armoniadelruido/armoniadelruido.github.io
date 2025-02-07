---
title: Survivor MySQL
date: 2025-01-24 0:31:00
categories: [BBDD,Postgress]
tags: [sysdamin, sql, postgress, psql, survivor]
---

service postgresql-9.6 initdb
service postgresql-9.6 start
chkconfig postgresql-9.6 on
directorio de instalacion:
/var/lib/pgsql/9.6/data
su - postgres
psql
DESS y PRE
create role documentum with login encrypted password 'Doc2018';

create database docdb owner documentum template template0 encoding 'UTF8' lc_collate 'en_US.UTF-8' lc_ctype 'en_US.UTF-8';
\q

PRO 

create role docboss with login encrypted password 'N4d4k3v3r';

create database docdb owner docboss template template0 encoding 'UTF8' lc_collate 'en_US.UTF-8' lc_ctype 'en_US.UTF-8';
\q

/var/lib/pgsql/9.6/data/postgresql.conf
aqui editar segun configuracfion deseada (decir a que ip estará escuchando)
#listen_addresses = 'localhost'         # what IP address(es) to listen on;
listen_addresses = '*'
                                        # comma-separated list of addresses;
                                        # defaults to 'localhost'; use '*' for all
                                        # (change requires restart)
#port = 5432                            # (change requires restart)
y luego, decir que bbdd, que usuario, y que ip podrá conectarse a la BBDD
/var/lib/pgsql/9.6/data/pg_hba.conf
# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
#host    all             all             127.0.0.1/32            ident\i
host    docdb           documentum      192.168.2.0/24           md5  (para que valide con credenciales suministradas))

Importante vigilar iptables
 /etc/sysconfig/iptables
 
Antes de aplicar revisar mejor configuración.

At any point in the config, so long as it is before the log and default rejection, add the line:
Opcion 1
iptables -A INPUT -p tcp -s 0/0 --sport 1024:65535 -d ip_cliente  --dport 5432 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp -s ip_cliente --sport 5432 -d 0/0  --dport 1024:65535 -m state --state ESTABLISHED -j ACCEPT

Opción 2
-A INPUT -s $SOURCE -p tcp --dport 5432 -j ACCEPT
Reiniciamos el servicio postgres

service postgresql-9.6 restart


ejecutar una sentencia sql con el usuario que queramos
Nos pedira el password del usuario, y entraremos a la BBDD como ese usuario para poder ejecutar un sql.
Entramos a la BBDD: psql -U "usuario" -d "nombrebbdd" -W
db=# \i or \include /path/filename

O tambien:
Por ejemplo desde el terminal lanzar: psql -d BBDD -a -f archivo.sql y nos pedira password






https://www.linuxito.com/gnu-linux/nivel-medio/893-configuracion-basica-de-un-servidor-postgresql
https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Virtualization/3.3/html/Installation_Guide/Preparing_a_Postgres_Database_Server_for_use_with_Red_Hat_Enterprise_Virtualization_Manager.html
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/managing_confined_services/sect-managing_confined_services-postgresql-configuration_examples
https://serverfault.com/questions/219682/connect-to-postgres-remotely-open-port-5432-for-postgres-in-iptables
http://desktop.arcgis.com/es/arcmap/10.3/manage-data/gdbs-in-postgresql/configure-postgresql-accept-connections.htm
http://www.linuxhispano.net/2011/02/15/permitir-conexiones-entrantes-a-un-servidor-postgresql/
