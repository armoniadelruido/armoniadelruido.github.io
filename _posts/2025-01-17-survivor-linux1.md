---
title: Survivor Linux1
date: 2025-01-17 0:31:00
categories: [Sistemas]
tags: [sysdamin, linux, yum, swap, lvm, tar, compress]
---

# Survivor básico 1 
Habra mas como este survivor, quizas cuando acabe de pasarlos todos a la web, los unifique.
Lanzar script al inicio:
```bash
Dentro de
/etc/rc.d/rc.local
poner la ruta del script
```
Instalar entorno gráfico:
```
mount -o loop /dev/cdrom /mnt
yum install -y xorg-x11-server-Xorg xorg-x11-xauth xorg-x11-apps
yum install gtk2 libXtst xorg-x11-fonts-Type1
yum groupinstall gnome-desktop x11 fonts
```
### YUM:
```bash
yum clean all
subscription-manager clean
```
Ver la lista de paquetes YUM:
```bash
yum list --noplugin
```
Actualiza el repositorio:
```bash
yum update --noplugin
```
Ampliar memoria swap:
```bash
swapoff -v /dev/mapper/rootvg-swap
lvm lvresize /dev/mapper/rootvg-swap -L+5G
mkswap /dev/mapper/rootvg-swap 
swapon -va
cat /proc/swaps

https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/5/html/deployment_guide/s1-swap-adding
```
### Ver memoria:
```bash
watch -n 1 free -m
```
Liberar RAM:
```bash
sync ; echo 3 > /proc/sys/vm/drop_caches ; echo "RAM Liberada"
```
Scriptillo
```bash
vi /usr/local/bin/nombre
chmod a+x /usr/local/bin/nombre
#!/bin/sh
# Limpiar es un script para limpiar la caché y liberar memoria
sync ; echo 3 > /proc/sys/vm/drop_caches ; echo "RAM Liberada"
```

Añadir fichero como swap:
```bash
dd if=/dev/zero of=swapfile bs=1024 count=1048576  count es igual a 1024 * tamaño deseado en Megas.  1 giga es igual a 1048576 2 Gigas = 2097152
chmod 0600 /swapfile
mkswap /swapfile
Activarlo
swapon /swapfile
```
Pasar la swap a la ram (revisar previamente si hay espacio)
```bash
swapoff -a ; swapon -a
cat /proc/swaps
```
### Discos

Ficheros de un directorio por peso:
```bash
du -sh
du -sk .[!.]* *| sort -n
du |sort -n
```
Cambiar tamaño particiones sistema deprecated cuando se crean mal los discos en el hipervisor:
```
 df -h
Filesystem                  Size  Used Avail Use% Mounted on
/dev/mapper/rootvg-root     2.9G  2.5G  429M  86% /
devtmpfs                    910M     0  910M   0% /dev
tmpfs                       920M     0  920M   0% /dev/shm
tmpfs                       920M  8.6M  912M   1% /run
tmpfs                       920M     0  920M   0% /sys/fs/cgroup
/dev/mapper/dadesvg-dades   477M  2.3M  445M   1% /dades
/dev/mapper/rootvg-var      2.0G  231M  1.7G  13% /var
/dev/mapper/rootvg-var_log  880M  162M  693M  19% /var/log
tmpfs                       184M     0  184M   0% /run/user/10326


[root@maquina usuario]# pvs
  PV         VG      Fmt  Attr PSize    PFree
  /dev/sda2  rootvg  lvm2 a--    <9.50g  <1.50g
  /dev/sdb1  dadesvg lvm2 a--  1020.00m 520.00m 
  
[root@maquina usuario]# lvs
  LV      VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  dades   dadesvg -wi-ao---- 500.00m
  root    rootvg  -wi-ao----   3.00g
  swap    rootvg  -wi-ao----   2.00g
  var     rootvg  -wi-ao----   2.00g
  var_log rootvg  -wi-ao----   1.00g
```

Ampliación del disco previa a la de la partición (para discos primarios).

Primero ampliar en el vmware.
Ahora nos vamos a la maquina.
```bash
 echo "- - -" > /sys/class/scsi_disk/X\:X\:X\:X/device/rescan  (las x son el numero de bus que nos asigna)
lsblk -S  nos dira en que BUS esta el disco para que pille el tamaño.
[root@maquina scsi_disk]# lsblk -S
NAME HCTL       TYPE VENDOR   MODEL             REV TRAN
sda  3:0:0:0    disk VMware   Virtual disk     2.0
sdb  3:0:1:0    disk VMware   Virtual disk     2.0
sr0  2:0:0:0    rom  NECVMWar VMware SATA CD00 1.00 sata
```
Asi que sería:
```bash
echo "- - -" > /sys/class/scsi_disk/3\:0\:0\:0/device/rescan
echo "- - -" > /sys/class/scsi_host/host0/scanning
```

Revisamos que el tamaño de disco ha cambiado, pero no la tabla de particiones.
```bash
[root@maquina ~]# fdisk -lu /dev/sda
```

Revisamos cual es el VG correcto.
```bash
[root@maquina ~]# vgs
  VG      #PV #LV #SN Attr   VSize    VFree
  dadesvg   1   1   0 wz--n- 1020.00m 520.00m
  rootvg    1   4   0 wz--n-    9.50g   1.50g
```

Y ahora nos vamos a editar la tabla de particiones:
```bash
fdisk /dev/sda

Command (m for help): p
Command (m for help): d seleccionamos cual eliminamos
Command (m for help): n hacemos una nueva
Select (default p): p si va a ser primaria o extendida
Partition number (2-4, default 2):  dejamos default
Command (m for help): t
Partition number (1,2, default 2): 2
Hex code (type L to list all codes):8e
Command (m for help): p
Command (m for help): w
```
Informamos del cambio en la tabla de particiones al kernel:
```bash
partx -u /dev/sda
pvresize /dev/sda2
```

Hay que saber qué sistema de ficheros utiliza la partición, y lo podemos ver con mount:
```bash
lsblk -f
mount | grep " $puntoDeMontaje " 

lvextend /dev/rootvg/root -L+500M
lvextend /dev/rootvg/var -L+500Mlvs

lvextend /dev/rootvg/var_log

/dev/mapper/rootvg-root on / type ext4 (rw,relatime,data=ordered)
resize2fs /dev/mapper/rootvg-root
```

### Grepeos de Logins
```bash
zgrep -hi "Failed password for " /var/log/secure* | sed "s/invalid user //" | tr -s " " | awk '{print $11" "$9}' | sort | uniq -c | sort -rn | head -20
```

### Comprobar recursos del sistema:
```bash
sar -rudpW -n DEV,SOCK
donde:
-r muestra uso de memoria
-u muestra uso de CPU
-d muestra uso de disco
-p imprime el nombre de los dispositivos de forma agradable (usado con -d)
-W muestra estadísticas de swap
-n DEV,SOCK muestra estadísticas de las interfaces de red y cantidad de sockets utilizados.

sar -s 16:30:00 -e 23:53:00 -r -n SOCK -f /var/log/sa/sa03
```

```bash
 grep -i client error_log | awk '{print $8}' | sort -u
```

### Comprimir/Descomprimir/Tartarizar

Archivos .tar.gz:
```bash
Empaquetar/Desempaquetar:
tar -czvf empaquetado.tar.gz /carpeta/a/empaquetar/
tar -xzvf archivo.tar.gz
```
Archivos .tar:
Empaquetar/Desempaquetar:
```bash
tar -cvf paquete.tar /dir/a/comprimir/
tar -xvf paquete.tar
```

Archivos .gz:
```bash
Comprimir/Descomprimir:
gzip -9 index.php
gzip -d index.php.gz

Archivos .zip:
Comprimir/Descomprimir:
```bash
zip archivo.zip carpeta
unzip archivo.zip
```

Quitar flag cambio pass despues de cambiarlo en aix:
```bash
vi /etc/security/paswd
pwdadm -c usuario
```



