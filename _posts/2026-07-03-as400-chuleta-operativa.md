---
title: AS400 chuleta operativa
date: 2026-07-03 01:10:00
categories: [Sistemas, Mainframe]
tags: [sysadmin, as400, ibmi, mainframe, operacion]
---

# AS400 chuleta operativa

Este post cubre operaciones basicas en AS400/IBM i: revision de sistema, jobs, colas de mensajes, spool, librerias y usuarios.

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
