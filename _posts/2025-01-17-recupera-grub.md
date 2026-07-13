---
title: Recuperar GRUB
date: 2025-01-17 0:31:00
categories: [Sistemas, Linux]
tags: [sysdamin, grub, rescue, lvm, luks, cryptosetup, cifrada]
---

# Como recuperar grub/sistema despúes de instalar windows junto a Linux en una partición cifrada.

Antes de instalar GRUB o ejecutar `fsck`, identifica bien disco, particion root, `/boot` y EFI con `lsblk -f`. Instala GRUB en el disco correcto, no en una particion al azar. `fsck` debe ejecutarse desde live/rescue o con el filesystem desmontado.

Secuencia para instalar el GRUB (desde una ISO live):

        Listar devices y particiones: sudo fdisk -l
        Montar particion root: sudo mount /dev/sdXY /mnt
        Mount particion boot (si hiciera falta): sudo mount /dev/sdXY /mnt/boot
        Mount particion EFI System (si hiciera falta): sudo mount /dev/sdXY /mnt/boot/efi
        Reinstalar GRUB: sudo grub-install --root-directory=/mnt /dev/sdX
        
Comandero, siendo sda5 la de root, sda2 la boot y sda1 la efi:
```bash
sudo mount /dev/sda5 /mnt
sudo mount /dev/sda2 /mnt/boot
sudo mount /dev/sda1 /mnt/boot/efi
sudo grub-install --root-directory=/mnt /dev/sda
```

Ahora activamos volumen que contenga la partición de root y montamos la partición:
```bash
vgdisplay
vgchange -ay NOMBRE
lsblk
cryptsetup open --type luks /dev/sda5 NOMBRE
ls /dev/mapper
mount /dev/mapper/NOMBRE-root /mnt/
mount /dev/sda5 /mnt/boot
grub-install --root-directory=/mnt /dev/sda
```

Secuencia para "bootear" desde la consola de GRUB:

        Listar devices y particiones: ls
        "Setear" la particion root: set root=[partition]
        Especificar archivo vmlinuz y la particion root: linux [vmlinuz file] root=/dev/sdXY
        Especificar initrd image: initrd [initrd image]
        "Bootar" el sistema: boot

Comandero:

```bash
set root=(hd0,gpt3))
linux /boot/vmlinuz root=/dev/sda5
initrd /boot/initrd.img
boot
```
            
Por ultimo, solo nos quedaría updatear grub:

```bash
 sudo update-grub
 ```

## Recuperacion con LUKS y LVM

```bash
cryptsetup luksOpen /dev/<DISCO_PARTICION> cryptroot
vgchange -ay
mount /dev/<VG>/<LV_ROOT> /mnt
mount /dev/<DISCO_BOOT> /mnt/boot
mount /dev/<DISCO_EFI> /mnt/boot/efi
for i in /dev /dev/pts /proc /sys /run; do mount --bind "$i" "/mnt$i"; done
chroot /mnt
grub-install /dev/<DISCO>
update-grub
```

## Reparar sistema de ficheros al arrancar

```bash
fsck -f /dev/<DISPOSITIVO>
mount -o remount,rw /
```

<!--
Fuentes consolidadas

- `grub_root`
- `reparar_grub`
- `recuperar grub sistema.txt`
- `Cifrar dispositivos con LUKS`
- `InstalarSO con cifrado`
- `fallo fsck error onboot`
-->
