---
title: Splunk chuleta operativa
date: 2026-07-03 00:40:00
categories: [Sistemas, Operación]
tags: [sysadmin, splunk, logs, monitoring, operacion]
---

# Splunk chuleta operativa

Este post sirve para lanzar busquedas Splunk, filtrar eventos, extraer campos y comprobar el estado del forwarder.

## Busquedas basicas

```text
index=<INDEX> sourcetype=<SOURCETYPE>
index=<INDEX> host=<HOSTNAME> ERROR
index=<INDEX> earliest=-1h latest=now
```

## Campos y agregaciones

```text
index=<INDEX> status>=500 | stats count by host, status
index=<INDEX> "<PATRON>" | timechart count by host
index=<INDEX> | top limit=20 source
```

## Extraccion rapida

```text
index=<INDEX> | rex field=_raw "usuario=(?<usuario>[^ ]+)" | stats count by usuario
```

## Forwarder

```bash
/opt/splunkforwarder/bin/splunk status
/opt/splunkforwarder/bin/splunk restart
/opt/splunkforwarder/bin/splunk list forward-server
```

## Revisar inputs

```bash
/opt/splunkforwarder/bin/splunk list monitor
/opt/splunkforwarder/bin/splunk btool inputs list --debug
```

<!--
Fuentes consolidadas

- `splunk.txt`
-->
