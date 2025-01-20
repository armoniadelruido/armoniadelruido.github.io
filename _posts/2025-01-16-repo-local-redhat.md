---
title: Repositorio local para RedHat
date: 2025-01-16 0:31:00
categories: [Sistemas, Red Hat]
tags: [sysdamin, update, upgrade, yum, reposync, createrepo, cves, parches, seguridad]
---

## Vamos a crear un servidor para tener nuestros repositorios local, nos permitirá ser mas ágiles para parchear y descargar actualizaciones, y mas cosas....

Nos suscribimos a red hat en el servidor que usaremos como repositorio:

```bash
subscription-manager clean
subscription-manager attach
```

Ponemos en el proxy (si tenemos alguno) en la config para llegar a redhat:
```bash
/etc/rhsm/rhsm.conf
vi /etc/yum.conf
```
```bash
proxy=http://proxy.MIDOMINIO.COM:9090
proxy_username=USUARIO
proxy_password=PASSWORD
```

Luego instalamos estos paquets:
```bash
yum install yum-utils createrepo httpd
```

Si tenemos algun firewall, abrimos puertos:
```bash
firewall-cmd --zone=public --permanent --add-service=http
firewall-cmd --zone=public --permanent --add-service=https
firewall-cmd --reload
```

Creamos los repositorios:

RHEL7:
```bash
reposync -g -l -d -m --repoid=rhel-7-server-extras-rpms --newest-only --download-metadata --download_path=/ruta/www/repos/
reposync -g -l -d -m --repoid=rhel-7-server-optional-rpms --newest-only --download-metadata --download_path=/ruta/www/repos/
reposync -g -l -d -m --repoid=rhel-7-server-rpms --newest-only --download-metadata --download_path=/ruta/www/repos/ 
reposync -g -l -d -m --repoid=rhel-7-server-supplementary-rpms --newest-only --download-metadata --download_path=/ruta/www/repos/
reposync -g -l -d -m --repoid=rhel-server-rhscl-7-rpms --newest-only --download-metadata --download_path=/ruta/www/repos/
```
RHEL6:
```bash
reposync -g -l -d -m --repoid=rhel-6-server-extras-rpms --newest-only --download-metadata --download_path=/ruta/www/repos/
reposync -g -l -d -m --repoid=rhel-6-server-optional-rpms --newest-only --download-metadata --download_path=/ruta/www/repos/
reposync -g -l -d -m --repoid=rhel-6-server-rh-common-rpms --newest-only --download-metadata --download_path=/ruta/www/repos/
reposync -g -l -d -m --repoid=rhel-6-server-rpms --newest-only --download-metadata --download_path=/ruta/www/repos/
reposync -g -l -d -m --repoid=rhel-6-server-supplementary-rpms --newest-only --download-metadata --download_path=/ruta/www/repos/
```
Crear Fichero con Metadatos:

RHEL7:
```bash
createrepo  /ruta/www/repos/rhel-7-server-extras-rpms/ 
createrepo  /ruta/www/repos/rhel-7-server-optional-rpms/ 
createrepo  /ruta/www/repos/rhel-7-server-rpms/
createrepo  /ruta/www/repos/rhel-7-server-supplementary-rpms/
createrepo  /ruta/www/repos/rhel-server-rhscl-7-rpms/
createrepo  /ruta/www/repos/rhel-secs/ -g comps.xml
```

RHEL6:
```bash
createrepo  /ruta/www/repos/rhel-6-server-extras-rpms/
createrepo  /ruta/www/repos/rhel-6-server-optional-rpms/
createrepo  /ruta/www/repos/rhel-6-server-rh-common-rpms/
createrepo  /ruta/www/repos/rhel-6-server-rpms/
createrepo  /ruta/www/repos/rhel-6-server-supplementary-rpms/
createrepo  /ruta/www/repos/rhel-secs/
```

Para actualizar los repositorios:

RHEL7:
```bash
createrepo --update /ruta/www/repos/rhel-7-server-extras-rpms/ 
createrepo --update /ruta/www/repos/rhel-7-server-optional-rpms/ 
createrepo --update /ruta/www/repos/rhel-7-server-rpms/
createrepo --update /ruta/www/repos/rhel-7-server-supplementary-rpms
createrepo --update /ruta/www/repos/rhel-server-rhscl-7-rpms/
createrepo --update /ruta/www/repos/rhel-secs/
```
RHEL6:
```bash
createrepo --update /ruta/www/repos/rhel-6-server-extras-rpms/
createrepo --update /ruta/www/repos/rhel-6-server-optional-rpms/
createrepo --update /ruta/www/repos/rhel-6-server-rh-common-rpms/
createrepo --update /ruta/www/repos/rhel-6-server-rpms/
createrepo --update /ruta/www/repos/rhel-6-server-supplementary-rpms/
createrepo --update /ruta/www/repos/rhel-secs/
```

### Ya tenemos un servidor con los repos de redhat en nuestra infrastructura local, ahora vamos a decirle a los clientes que lo usen para actualizar sus repos:

En los clientes RHEL7:
```bash
subscription-manager clean
subscription-manager config --server.hostname=rhel7-repos-locales.MIDOMINIO.COM
```
```bash
vi /etc/rhsm/rhsm.conf
baseurl = https://rhel7-repos-locales.MIDOMINIO.COM
```
```bash
vi /etc/yum.repos.d/redhat7_local.repo

[rhel-7-server-extras-rpms]
name=RHEL7 extras
baseurl=https://rhel7-repos-locales.MIDOMINIO.COM/rhel-7-server-extras-rpms/
gpgcheck=0
sslverify=0
enabled=1

[rhel-7-server-optional-rpms]
name=RHEL7 optional
baseurl=https://rhel7-repos-locales.MIDOMINIO.COM/rhel-7-server-optional-rpms/
gpgcheck=0
sslverify=0
enabled=1

[rhel-7-server-rpms]
name=RHEL7 server
baseurl=https://rhel7-repos-locales.MIDOMINIO.COM/rhel-7-server-rpms/
gpgcheck=0
sslverify=0
enabled=1

[rhel-7-server-supplementary-rpms]
name=RHEL7 supplementary
baseurl=https://rhel7-repos-locales.MIDOMINIO.COM/rhel-7-server-supplementary-rpms/
gpgcheck=0
sslverify=0
enabled=1

[rhel-server-rhscl-7-rpms]
name=RHEL7 supplementary
baseurl=https://rhel7-repos-locales.MIDOMINIO.COM/rhel-server-rhscl-7-rpms/
gpgcheck=0
sslverify=0
enabled=1


[rhel-secs]
name=RHEL7 sec
baseurl=https://rhel7-repos-locales.MIDOMINIO.COM/rhel-secs/
gpgcheck=0
sslverify=0
enabled=1
```

En los clientes RHEL6:
```bash
subscription-manager clean
subscription-manager config --server.hostname=rhelrepos6x64.MIDOMINIO.COM
```
```bash
vi /etc/yum.repos.d/redhat6_local.repo


[rhel-6-server-extras-rpms]
name=RHEL6 extras
baseurl=https://rhelrepos6x64.MIDOMINIO.COM/rhel-6-server-extras-rpms/
gpgcheck=0
sslverify=0
enabled=1

[rhel-6-server-optional-rpms]
name=RHEL6 optional
baseurl=https://rhelrepos6x64.MIDOMINIO.COM/rhel-6-server-optional-rpms/
gpgcheck=0
sslverify=0
enabled=1

[rhel-6-server-rh-common-rpms]
name=RHEL6 server
baseurl=https://rhelrepos6x64.MIDOMINIO.COM/rhel-6-server-rh-common-rpms/
gpgcheck=0
sslverify=0
enabled=1

[rhel-6-server-rpms]
name=RHEL6 supplementary
baseurl=https://rhelrepos6x64.MIDOMINIO.COM/rhel-6-server-rpms/
gpgcheck=0
sslverify=0
enabled=1

[rhel-6-server-supplementary-rpms]
name=RHEL6 supplementary
baseurl=https://rhelrepos6x64.MIDOMINIO.COM/rhel-6-server-supplementary-rpms/
gpgcheck=0
sslverify=0
enabled=1


[rhel-secs]
name=RHEL6 sec
baseurl=https://rhelrepos6x64.MIDOMINIO.COM/rhel-secs/
gpgcheck=0
sslverify=0
enabled=1
```

## Respositorios de seguridad

En la maquina servidor:
```bash
yum -y install yum-plugin-security
yum updateinfo list available
yum list-sec 
yum updateinfo list sec    - solo muestra las RHSA- que falten parchear en la maquina
yum updateinfo list cves   - solo muestra las CVES- que falten parchear en la maquina.
yum updateinfo list security all
yum updateinfo list security all |grep -v "i"  - muestra lo que falta por updatar de seguridad
yum --security check-update
yum update --downloadonly --security --downloaddir=/ruta/prova
yum info-sec
yum updateinfo RHSA-2019:0163
yum updateinfo list bugfix - solo muestra las RHBA- que falen parchear en la maquina
yum updateinfo summary
```


### Comandos útiles para updatear securizando:

Instalar seguridad mas todas las erratas:
```bash
yum update --security 
```
Solo seguridad:
```bash
yum update-minimal --security 
```
Instalar por cve:
```bash
yum update --cve CVE-2008-0947
```
Instalar por erratas:
```bash
yum update --advisory=RHBA-2018:0739
```
Ver los RSHA instalados:
```bash
yum updateinfo list security installed
```
Ver diferentes versiones disponibles
```bash
yum --showduplicates list 
```

Revisar esta url si deja de funcionar:

https://access.redhat.com/solutions/23016#security

Info sobre que es cada errata:
```bash
https://access.redhat.com/articles/2130961
Red Hat Security Advisory (RHSA): Actualizaciones de seguridad, pueden contener bugfixes o enancements.
Red Hat Bug Advisory (RHBA): Bugfixes y pueden contener enancements, pero nunca pueden contener fixes de seguridad.
Red Hat Enhancement Advisory (RHEA): Enancements o new features, pero nunca contienenen bugfixes o security fixes. SI un paquete es actualizado con una nueva funcionalidad aparece aqui.

1 mes critical
3 meses high
5 meses moderate
```

Ver paquetes instalados
```bash
rpm -qa --last
yum list installed
```
Para ver si estamos afectados por una vulnerabilidad, primero buscamos el paquet en el repo para ver su nombre:
```bash
yum search all SDL-
```
luego lo buscamos, y si no machea, es que no es necesario hacer nada:
```bash
rpm -qa --last|grep -i sdl
```

### Creamos el directorio donde alojaremos los paquetes de seguridad:
Luego copiamos el *updateinfo.xml.gz del cache de yum al directorio que aloja los repos:
```bash
cp /var/cache/yum/x86_64/7Server/rhel-7-server-rpms/*updateinfo.xml.gz /ruta/www/repos/rhel-secs/repodata
gzip -d /ruta/www/repos/rhel-secs/repodata/*-updateinfo.xml.gz 
mv *-updateinfo.xml /ruta/www/repos/rhel-secs/repodata/updateinfo.xml
cp /var/cache/yum/x86_64/7Server/rhel-7-server-rpms/repomd.xml /ruta/www/repos/rhel-secs/repodata/
modifyrepo /ruta/www/repos/rhel-secs/repodata/updateinfo.xml /ruta/www/repos/rhel-secs/repodata/
createrepo /ruta/www/repos/rhel-secs/

cp /var/cache/yum/x86_64/6Server/rhel-6-server-rpms/*updateinfo.xml.gz /ruta/www/repos/rhel-secs/repodata/
gzip -d /ruta/www/repos/rhel-secs/repodata/*-updateinfo.xml.gz
mv *-updateinfo.xml /ruta/www/repos/rhel-secs/repodata/updateinfo.xml
cp /var/cache/yum/x86_64/6Server/rhel-6-server-rpms/repodata/repomd.xml /ruta/www/repos/rhel-secs/repodata/
modifyrepo /ruta/www/repos/rhel-secs/repodata/updateinfo.xml /ruta/www/repos/rhel-secs/repodata/
```

###########################################

```bash
find /var/cache/yum/ -name updateinfo.xml

cp -f /var/cache/yum/x86_64/7Server/rhel-7-server-rpms/gen/updateinfo.xml /ruta/www/repos/rhel-secs/repodata/updateinfo.xml
cp -f /var/cache/yum/x86_64/7Server/rhel-7-server-rpms/repomd.xml /ruta/www/repos/rhel-secs/repodata/
cp -f /var/cache/yum/x86_64/7Server/rhel-7-server-rpms/*-primary.sqlite.bz2 /ruta/www/repos/rhel-secs/repodata/
cp -f /var/cache/yum/x86_64/7Server/rhel-7-server-rpms/*-comps.xml /ruta/www/repos/rhel-secs/repodata/
modifyrepo /ruta/www/repos/rhel-secs/repodata/updateinfo.xml /ruta/www/repos/rhel-secs/repodata/
```
## Bonus
### Script para automatizarlo.

Dejo un esqueleto de script en nsh, que nos puede ayudar a empezar las tareas de automatización de updates de seguridad.

```bash
#!/usr/bin/nsh
for i in `cat /tools/scripts/host_list`
do
echo $i
#nexec $i uname -a
nexec $i yum updateinfo list cves > /tools/scripts/reports/$i
done
```

Cajón desastre:
```bash
vi /usr/lib/rsc/exports

10.0.0.1   rw,user=root
MAQUINA.MIDOMINIO.COM rw,user=root

/ruta/www/repos/rhel-dvd-7_0-rpms/


./var/tmp/createrepomGdE3M/garbageid/0b1f07d77bafde723090080278693ac38af894407ffc718dc21551c168e60d9f-filelists.sqlite.bz2
./var/tmp/createrepoPM_fSd/garbageid/44d520fa4a03fc713724f4711ce4f3e37298799e9ed60f58f5bf6a914655b62d-filelists.sqlite.bz2
./ruta/www/repos/rhel-server-rhscl-7-rpms/repodata/1ab3568eae4d289aa106994b438d321a2bbb6964dec32e8aec23b22ec3e121e0-filelists.sqlite.bz2
./ruta/www/repos/rhel-7-server-supplementary-rpms/repodata/44d520fa4a03fc713724f4711ce4f3e37298799e9ed60f58f5bf6a914655b62d-filelists.sqlite.bz2
./ruta/www/repos/rhel-7-server-rpms/repodata/0b1f07d77bafde723090080278693ac38af894407ffc718dc21551c168e60d9f-filelists.sqlite.bz2
./ruta/www/repos/rhel-7-server-extras-rpms/repodata/00d100f6401d746438aa47de14df910d9a422faa4f5dc6d1d2d7d21ec6296a02-filelists.sqlite.bz2
./ruta/www/repos/rhel-7-server-optional-rpms/repodata/8f4c9e2497da313a07307f3187aa128e5b5c1550f8f81f74ae567b5e033a6ab4-filelists.sqlite.bz2
./ruta/www/repos/rhel-secs/repodata/01a3b489a465bcac22a43492163df43451dc6ce47d27f66de289756b91635523-filelists.sqlite.bz2
```

## Bonus 2
### Optimizamos la maquina virtual para operar en vmware con las vmware-tools.

```bash
 /etc/dracut.conf.d/
 in nvdimm-security.conff
 add_drivers+=" libnvdimm "
 in vmware-tools.conf
 add_drivers+=" vmxnet3 vmw_pvscsi "
  dracut -f -v
```
Ver si necesitamos restart:
```bash
needs-restarting -r
```

 

