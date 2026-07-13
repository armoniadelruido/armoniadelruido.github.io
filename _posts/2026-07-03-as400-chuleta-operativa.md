---
title: AS400 chuleta operativa
date: 2026-07-03 01:10:00
categories: [Sistemas, Mainframe]
tags: [sysadmin, as400, ibmi, mainframe, operacion]
---

# AS400 chuleta operativa

Este post cubre operaciones basicas en AS400/IBM i: revision de sistema, jobs, colas de mensajes, spool, librerias y usuarios.

La mayoria de comandos de esta chuleta son de consulta, pero muchas pantallas permiten lanzar acciones desde menus. Usalos para mirar estado y diagnosticar; si vas a modificar algo, confirma procedimiento, autorizacion y objeto exacto.

| Comando | Para que sirve | Cuando usarlo |
|---|---|---|
| `WRKSYSSTS` | Estado general del sistema | Primera revision de carga y recursos |
| `WRKACTJOB` | Jobs activos | Ver procesos consumiendo CPU o esperando |
| `WRKJOB` | Detalle de un job | Investigar un job concreto |
| `DSPJOBLOG` | Log de job | Revisar errores o mensajes |
| `WRKUSRPRF` | Perfiles de usuario | Revisar usuarios con cuidado antes de modificar |

## Sistema y jobs

```text
WRKSYSSTS
WRKACTJOB
WRKJOB <JOB>
DSPJOBLOG JOB(<JOB>)
```

## Colas y mensajes

```text
WRKMSGQ
DSPMSG <MSGQ>
WRKOUTQ
WRKSPLF
```

## Librerias y objetos

```text
WRKLIB <LIBRERIA>
WRKOBJ <LIBRERIA>/*ALL
DSPOBJD OBJ(<LIBRERIA>/<OBJETO>) OBJTYPE(*ALL)
```

## Usuarios

```text
WRKUSRPRF <USUARIO>
DSPUSRPRF <USUARIO>
```

<!--
Fuentes consolidadas

- `as400.txt`
-->
