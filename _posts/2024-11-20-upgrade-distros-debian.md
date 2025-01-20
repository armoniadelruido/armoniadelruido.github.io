---
title: Upgrades Debian
date: 2024-11-29 0:31:00
categories: [Sistemas, Debian]
tags: [sysdamin, debian, apt, sed, updates]
---

# Vamos a pasar de una version de debian a otra.  

Actualizamos nuestro sistema primero de todo:
```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get full-upgrade
sudo apt-get --purge autoremove
```

Reiniciamos.

Despues de reiniciar, vamos a cambiar la distro, en este caso pasaremos de bullseye a bookworm:
```bash
cp -v /etc/apt/sources.list /root/sources.list-bakup.11.bullseye
vim /etc/apt/sources.list
```
Podemos sustituir directamente y hacer una copia de seguridad con:
```bash
sudo sed -i'.bak' 's/bullseye/bookworm/g' /etc/apt/sources.list
```
Hacemos un update para que pille los repos, y luego un upgrade minimo:
```bash
apt-get update
apt-get upgrade --without-new-pkgs
```

Luego de responder las preguntas, hacemos el full upgrade:
```bash
apt-get full-upgrade
```

Y reiniciamos
```bash
systemctl reboot
```
Revisamos nuestra version de sistema:

```bash
uname -mrs
lsb_release -a
```

Cuando estemos seguros de que todo esta bien, podemos purgar restos:
```bash
apt-get --purge autoremove
```

