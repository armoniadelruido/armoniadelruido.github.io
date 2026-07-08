---
title: AIX Power HACMP y LPAR
date: 2026-07-03 00:30:00
categories: [Sistemas, Aix]
tags: [sysadmin, aix, power, hacmp, powerha, lpar]
---

# AIX Power HACMP y LPAR

Este post ayuda a diagnosticar AIX, revisar errores, comprobar PowerHA/HACMP, validar discos y obtener datos basicos de LPAR.

## Errores del sistema

```bash
errpt
errpt -a
errpt -a | more
```

## PowerHA/HACMP

```bash
clstat
cldump
clRGinfo
clshowsrv -v
```

## Recursos y discos

```bash
lspv
lsvg
lsvg -p <VG>
lsvg -l <VG>
lspath
```

## Upgrade y paquetes

```bash
oslevel -s
instfix -i | grep ML
lslpp -L | grep <PAQUETE>
installp -acgXd /ruta/ejemplo <PAQUETE>
```

## LPAR y monitorizacion

```bash
lparstat -i
vmstat 2 10
iostat 2 10
```

<!--
Fuentes consolidadas

- `hacmp.txt`
- `upgrades_aix.txt`
- `errpt.txt`
- `lpar2rrd`
- `powerlabdes.txt`
- `movimiento_p9.txt`
- `control_FS_nagios_aix.txt`
- `new 30.txt`
- `new 32.txt`
- `new 39.txt`
- `new 40.txt`
- `new 90.txt`
-->
