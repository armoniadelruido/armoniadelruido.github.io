---
title: Clonar Devices
date: 2024-11-26 0:31:00
categories: [Sistemas]
tags: [armbian, sd, dd, clone]
---

# Pues aqui vamos a hablar de administracion del SO en general, y si el post se hace muy largo, tendremos que hacer mas.  
Para crear una IMG nos ponemos en el directorio donde esta la *.img

```bash
cd /ruta/imagenes
blkid
```
Nos aparecen los devices montados. Identificamos el nuestro y lo desmontamos con umount (del dev entero, no las particiones del /dev
Si nos sale /dev/sdc1, pues seria /sdc.

```bash
umount /dev/sdc
```
luego:
```bash
dd bs=4M if=pi3_1.img of=/dev/sdb
```
Si queremos hacer una copia de la sd al disco duro es al reves:
```bash
dd bs=4M if=/dev/sd* of=/media/collar/IMGs/makemyself/NOMBREIMAGEN.img
```
Así de simple  :)
Y si es un DIsco duro completo????
La sintaxis más básica, sería ésta [como root]:
```bash
dd if=[origen] of=[destino] status=progress
```
Por lo que si quisiéramos clonar un disco duro:
```bash
dd if=/dev/sd* (origen) of=/dev/sd* (destino) bs=1M status=progress 
dd if=/dev/sdb of=/dev/nvme0n1 bs=1M status=progress 
```
Hacemos un lsblk en nuestra Raspberry o placa similar para ver identificar la nand integrada:
```bash
root@pihole:~# lsblk
NAME         MAJ:MIN RM    SIZE RO TYPE MOUNTPOINTS
sda            8:0    0      0B  0 disk 
mmcblk0      179:0    0   14.8G  0 disk 
└─mmcblk0p1  179:1    0   10.7G  0 part /
mmcblk2      179:8    0   14.6G  0 disk 
└─mmcblk2p1  179:9    0   13.6G  0 part 
mmcblk2boot0 179:16   0      4M  1 disk 
mmcblk2boot1 179:24   0      4M  1 disk 
zram0        253:0    0 1005.9M  0 disk [SWAP]
zram1        253:1    0     50M  0 disk 
zram2        253:2    0      0B  0 disk 
root@pihole:~# 

```
mmcblk2 es EMMC, y mmcblk0 es SD)

De EMMC a SD
```bash
dd if=/dev/mmcblk2 of=/dev/mmcblk0 bs=1M status=progress
```
De SD a EMMC
```bash
dd if=/dev/mmcblk0 of=/dev/mmcblk2 bs=1M status=progress
```
Pero si nos fijamos bien, el tamaño total de la nand (mmcblk2 - 14.6G), es ligeramente inferior que el de nuestra sd (mmcblk0 - 14.8G), asi que en lugar de la sd completa, usaremos la particion para clonarla, que es mas pequeña y entrará:
```bash
dd if=/dev/mmcblk0p1 of=/dev/mmcblk2p1 bs=1M status=progress
```

Con bs=1M, estamos diciendo que tanto la lectura como la escritura se haga en bloques de 1 megabyte (menos, sería más lento pero más seguro, y con más nos arriesgamos a perder datos por el camino).
timestamp para 500gb es de 240 minutos

```bash
root@pihole:~# dd if=/dev/mmcblk0 of=/dev/mmcblk2 bs=1M status=progress
15631122432 bytes (16 GB, 15 GiB) copied, 1302 s, 12.0 MB/s
15634268160 bytes (16 GB, 16 GiB) copied, 1313.87 s, 11.9 MB/s
root@pihole:~# 
```

Y ahora cuando acabe solo nos falta decirle que arranque desde la nand en lugar de la ssd.
En este caso, mi pihole esta en una placa orange, y no en una raspberry, asi que lo hacemos con armbian-config

![My helpful screenshot](/assets/images/clonar-devices/armbian-config1.png)
![My helpful screenshot](/assets/images/clonar-devices/armbian-config2.png)
![My helpful screenshot](/assets/images/clonar-devices/armbian-config3.png)

Con esto ya podemos sacar la sd, y arrancar desde la nand integrada.

Ahora podriamos usar la ssd o bien como mas almacenamiento (cosa que en mi caso no me interesa) o bien para ir sincronizando ambos devices, y asi tener una copia de respaldo en caso de disaster recovery.

```bash

```



