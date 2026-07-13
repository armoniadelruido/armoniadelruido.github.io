---
title: Airgeddon auditoria WiFi
date: 2026-07-05 00:20:00
categories: [Sistemas, Seguridad]
tags: [airgeddon, wifi, auditoria, seguridad, wireless]
---

# Airgeddon auditoria WiFi

Este post resume requisitos y comprobaciones previas para usar Airgeddon en auditorias WiFi controladas y autorizadas.

## Uso responsable

Airgeddon debe usarse solo en redes propias o entornos donde exista autorizacion explicita para realizar pruebas.

El modo monitor y `airmon-ng check kill` pueden cortar la WiFi del propio equipo. Si estas conectado por esa interfaz, perderas la sesion. Mejor usar consola local o una segunda conexion, y al terminar vuelve a modo gestionado y reinicia NetworkManager si hace falta.

## Requisitos habituales

```bash
airmon-ng
airodump-ng
aireplay-ng
aircrack-ng
iwconfig
ip
```

## Comprobar adaptador WiFi

```bash
iw dev
ip link
airmon-ng
```

## Modo monitor

```bash
airmon-ng check kill
airmon-ng start <INTERFAZ>
```

## Lanzar Airgeddon

```bash
./airgeddon.sh
```

## Volver a modo gestionado

```bash
airmon-ng stop <INTERFAZ_MONITOR>
systemctl restart NetworkManager
```

## Problemas comunes

- Adaptador sin soporte de modo monitor.
- Driver incompatible o firmware ausente.
- NetworkManager bloqueando la interfaz.
- Falta de herramientas opcionales para ataques o capturas concretas.

## Vendor local

Existe una copia local del proyecto bajo `/tools/scripts/airgeddon/`. Para documentacion publica es mejor referenciar comandos de uso y requisitos, no volcar codigo del proyecto externo.

<!--
Fuentes consolidadas

- `/tools/scripts/airgeddon/README.md`
- `/tools/scripts/airgeddon/CHANGELOG.md`
- `/tools/scripts/airgeddon/CONTRIBUTING.md`
- `/tools/scripts/airgeddon/CODE_OF_CONDUCT.md`
- `/tools/scripts/airgeddon/LICENSE`
-->
