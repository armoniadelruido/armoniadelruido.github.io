---
title: Raspberry Orange Pi y servicios homelab
date: 2026-07-04 00:40:00
categories: [Homelab, SBC]
tags: [raspberry, orangepi, armbian, pihole, wordpress, nextcloud, homelab]
---

# Raspberry Orange Pi y servicios homelab

Este post cubre ajustes de Raspberry/Orange Pi, preparacion de servicios web ligeros, WiFi, WordPress, Nextcloud y optimizaciones para alargar la vida de la SD.

En placas pequeñas, quitar swap o desactivar servicios puede mejorar vida de SD y consumo, pero tambien puede dejar el sistema sin margen si falta RAM o si dependes de descubrimiento de red, perifericos o gestion grafica. Desactiva solo lo que sepas que no usas y prueba tras reiniciar.

## `/boot/config.txt`

```ini
dtparam=audio=on
gpu_mem=128
start_x=1
max_usb_current=1
hdmi_ignore_cec_init=1
```

## Optimizar Raspberry Pi

```bash
apt update
apt install -y rpi-eeprom
```

Desactivar swap si no es necesario:

```bash
dphys-swapfile swapoff
dphys-swapfile uninstall
update-rc.d dphys-swapfile remove
apt purge -y dphys-swapfile
```

Usar tmpfs para `/tmp`:

```bash
cp /usr/share/systemd/tmp.mount /etc/systemd/system/tmp.mount
systemctl enable tmp.mount
systemctl start tmp.mount
```

## Servicios prescindibles

```bash
systemctl disable avahi-daemon triggerhappy packagekit
systemctl stop avahi-daemon triggerhappy packagekit
apt purge -y avahi-daemon triggerhappy packagekit
```

## WiFi con `wpa_supplicant`

```text
country=ES
network={
    ssid="<SSID>"
    psk="<PASSWORD>"
}
```

## WordPress basico

```bash
apt install -y apache2 php mariadb-server php-mysql
cd /var/www/html
wget https://wordpress.org/latest.tar.gz
tar xzf latest.tar.gz
rsync -a wordpress/ ./
chown -R www-data:www-data /var/www/html
```

## Preparar imagen reutilizable

Cuando la base tenga paquetes, PHP, BBDD y webserver configurados, conviene clonar la SD para reutilizarla como plantilla.

<!--
Fuentes consolidadas

- `clonar_sd`
- `oragepipcplus`
- `orangepipcplusV2`
- `ubuntu_xenial_orangepi_plus2`
- `fruitnanny`
- `magicmirror`
- `pihole`
- `realtek_8812au`
- `Raspberrys/discousbraspberry`
- `Raspberrys/optimizar_pi4`
- `Raspberrys/pi_server_lite_php7_msql`
- `Raspberrys/raspberry_config_txt`
- `Raspberrys/Raspberry_DNs`
- `Raspberrys/warberry`
- `Raspberrys/WIFIS_RASPBIANES`
- `Raspberrys/wordpresspi`
- `Raspberrys/wordpresspi_php7`
-->
