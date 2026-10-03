---
title: "Cifrar discos con LUKS: creación, claves y recuperación"
date: 2026-09-19 09:00:00
categories: [Sistemas, Linux]
tags: [sysadmin, linux, luks, cryptsetup, cifrado]
---

# Cifrar discos con LUKS: creación, claves y recuperación

LUKS cifra un dispositivo de bloques y permite administrar varias frases de paso mediante *keyslots*. Esta guía usa identificadores genéricos de forma deliberada: antes de ejecutar cualquier orden hay que sustituir `<DISPOSITIVO>` por el disco correcto y confirmarlo con más de una comprobación.

> `luksFormat` destruye el acceso al contenido anterior del dispositivo. No se debe ejecutar sobre un disco montado, un volumen en uso ni una unidad cuyo contenido no tenga copia verificada.

## Preparación

```bash
sudo apt install cryptsetup
lsblk -o NAME,SIZE,FSTYPE,LABEL,UUID,MOUNTPOINTS,MODEL
sudo blkid <DISPOSITIVO>
findmnt --source <DISPOSITIVO>
```

El dispositivo debe estar desmontado. En los ejemplos:

```text
<DISPOSITIVO> = /dev/<DISCO_PARTICION>
<MAPEO>       = volumen_cifrado
<MONTAJE>     = /mnt/volumen_cifrado
```

## Crear el contenedor LUKS2

```bash
sudo cryptsetup --type luks2 luksFormat <DISPOSITIVO>
sudo cryptsetup open <DISPOSITIVO> <MAPEO>
sudo mkfs.ext4 -L <ETIQUETA> /dev/mapper/<MAPEO>
sudo mkdir -p <MONTAJE>
sudo mount /dev/mapper/<MAPEO> <MONTAJE>
```

Comprobaciones:

```bash
lsblk -f
findmnt <MONTAJE>
sudo cryptsetup status <MAPEO>
```

Para desmontar y cerrar:

```bash
sudo umount <MONTAJE>
sudo cryptsetup close <MAPEO>
```

## Añadir una segunda frase de paso

LUKS permite mantener varias credenciales independientes. Al añadir una nueva, primero solicita una frase válida existente y después la nueva.

```bash
sudo cryptsetup luksDump <DISPOSITIVO>
sudo cryptsetup luksAddKey <DISPOSITIVO>
sudo cryptsetup luksDump <DISPOSITIVO>
```

Antes de retirar una clave antigua, hay que probar la nueva en una apertura real:

```bash
sudo cryptsetup open <DISPOSITIVO> <MAPEO_PRUEBA>
sudo cryptsetup close <MAPEO_PRUEBA>
```

Después puede eliminarse una frase concreta:

```bash
sudo cryptsetup luksRemoveKey <DISPOSITIVO>
```

Nunca se debe borrar el último *keyslot* utilizable.

## Copia del encabezado

Si se daña el encabezado LUKS, los datos pueden quedar inaccesibles aunque la zona cifrada siga intacta. Conviene guardar una copia en un medio distinto y protegido:

```bash
sudo cryptsetup luksHeaderBackup <DISPOSITIVO> \
  --header-backup-file <RUTA_SEGURA>/cabecera-luks.img
sudo chmod 600 <RUTA_SEGURA>/cabecera-luks.img
```

La copia del encabezado es sensible: junto con una frase válida puede facilitar el acceso al volumen y además conserva el estado de los *keyslots* del momento en que se creó.

Una restauración solo debe hacerse cuando se ha confirmado que el encabezado está dañado y existe una imagen correcta del dispositivo:

```bash
sudo cryptsetup luksHeaderRestore <DISPOSITIVO> \
  --header-backup-file <RUTA_SEGURA>/cabecera-luks.img
```

## Cifrado durante la instalación del sistema

Para un sistema Debian con cifrado completo, el esquema habitual es:

1. Una partición EFI o `/boot` según el modo de arranque y el diseño elegido.
2. Un contenedor LUKS para el resto del espacio.
3. LVM dentro de LUKS para separar `/`, `/home` y `swap` si se necesita flexibilidad.
4. Una frase de paso robusta y un medio de recuperación probado.

El instalador puede crear esta estructura automáticamente con el particionado guiado cifrado. Antes de instalar, se debe comprobar el modo BIOS/UEFI y la política de arranque del equipo.

## Lista de control

- Confirmar el dispositivo por tamaño, modelo y punto de montaje.
- Tener copia de los datos antes de formatear.
- Probar cada nueva frase antes de retirar otra.
- Guardar el encabezado fuera del propio disco.
- Documentar el procedimiento de desbloqueo sin almacenar las frases.
- Probar que las copias de seguridad pueden restaurarse.

## Referencias

- [Manual de `cryptsetup`](https://manpages.debian.org/unstable/cryptsetup-bin/cryptsetup.8.en.html)
- [Manual de `luksAddKey`](https://manpages.debian.org/trixie/cryptsetup-bin/cryptsetup-luksAddKey.8.en.html)
