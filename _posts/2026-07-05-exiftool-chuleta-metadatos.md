---
title: ExifTool chuleta de metadatos
date: 2026-07-05 00:10:00
categories: [Sistemas, Herramientas]
tags: [exiftool, metadatos, imagenes, pdf, cli]
---

# ExifTool chuleta de metadatos

Este post sirve para consultar, exportar, modificar y limpiar metadatos de ficheros usando ExifTool desde terminal.

## Ver metadatos

```bash
exiftool fichero.jpg
exiftool -a -u -g1 fichero.jpg
```

## Extraer campos concretos

```bash
exiftool -DateTimeOriginal -CreateDate -ModifyDate fichero.jpg
exiftool -GPSLatitude -GPSLongitude fichero.jpg
```

## Exportar a JSON o CSV

```bash
exiftool -json fichero.jpg > metadatos.json
exiftool -csv *.jpg > metadatos.csv
```

## Borrar metadatos

```bash
exiftool -all= fichero.jpg
```

Conservar el original desactivando backup automatico:

```bash
exiftool -overwrite_original -all= fichero.jpg
```

## Cambiar fechas

```bash
exiftool -DateTimeOriginal="2026:07:05 12:00:00" fichero.jpg
exiftool "-AllDates+=0:0:0 1:30:0" fichero.jpg
```

## Procesar directorios

```bash
exiftool -r -ext jpg -ext png /ruta/imagenes
exiftool -r -overwrite_original -all= /ruta/imagenes
```

## Usar ficheros de argumentos

```bash
exiftool -@ reglas.args fichero.jpg
```

<!--
Fuentes consolidadas

- `/tools/scripts/exiftool/README`
- `/tools/scripts/exiftool/lib/Image/ExifTool/README`
- `/tools/scripts/exiftool/blib/lib/Image/ExifTool/README`
- `/tools/scripts/exiftool/arg_files/exif2iptc.args`
- `/tools/scripts/exiftool/arg_files/xmp2gps.args`
- `/tools/scripts/exiftool/arg_files/iptc2exif.args`
- `/tools/scripts/exiftool/arg_files/exif2xmp.args`
- `/tools/scripts/exiftool/arg_files/xmp2pdf.args`
- `/tools/scripts/exiftool/arg_files/xmp2iptc.args`
- `/tools/scripts/exiftool/arg_files/iptcCore.args`
- `/tools/scripts/exiftool/arg_files/gps2xmp.args`
- `/tools/scripts/exiftool/arg_files/pdf2xmp.args`
- `/tools/scripts/exiftool/arg_files/xmp2exif.args`
- `/tools/scripts/exiftool/arg_files/iptc2xmp.args`
-->
