---
title: IBM MQ colas y brokers
date: 2026-07-03 00:10:00
categories: [Sistemas, Middleware]
tags: [sysadmin, ibm-mq, mq, colas, brokers, middleware]
---

# IBM MQ colas y brokers

Este post sirve para revisar gestores de colas, consultar profundidad de colas, arrancar/parar canales y hacer comprobaciones basicas de brokers.

## Gestores de colas

```bash
dspmq
strmqm <QMGR>
endmqm <QMGR>
runmqsc <QMGR>
```

## Consultar colas

```text
DISPLAY QLOCAL(*) CURDEPTH MAXDEPTH
DISPLAY QLOCAL(<COLA>) ALL
DISPLAY QSTATUS(<COLA>) TYPE(QUEUE) ALL
```

## Canales

```text
DISPLAY CHANNEL(*) CHLTYPE
DISPLAY CHSTATUS(*)
START CHANNEL(<CANAL>)
STOP CHANNEL(<CANAL>)
```

## Purga controlada de cola

Antes de purgar, confirmar que no hay consumidores activos y que el contenido no es recuperable.

```text
CLEAR QLOCAL(<COLA>)
```

## Brokers

```bash
mqsilist
mqsistart <BROKER>
mqsistop <BROKER>
mqsireportproperties <BROKER> -a
```

<!--
Fuentes consolidadas

- `AC_nuevo_QM.txt`
- `colasMQ.txt`
- `colas_mq.txt`
- `brokers.txt`
- `nuevo_mq.txt`
- `purge_colas.txt`
-->
