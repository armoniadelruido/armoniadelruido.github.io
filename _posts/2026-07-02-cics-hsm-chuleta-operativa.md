---
title: CICS y HSM chuleta operativa
date: 2026-07-02 01:30:00
categories: [Sistemas, Mainframe]
tags: [sysadmin, cics, hsm, tpv, operacion]
---

# CICS y HSM chuleta operativa

Este post cubre consultas CICS, acciones sobre terminales y transacciones, revision de logs HSM e incidencias transaccionales.

## CICS: comandos de consulta

```text
CEMT I TASK
CEMT I TRAN(<TRANSACCION>)
CEMT I TERM(<TERMINAL>)
CEMT I FILE(<FICHERO>)
```

## CICS: acciones habituales

```text
CEMT S TRAN(<TRANSACCION>) ENABLED
CEMT S TRAN(<TRANSACCION>) DISABLED
CEMT S TERM(<TERMINAL>) INSERVICE
CEMT S TERM(<TERMINAL>) OUTSERVICE
```

## DB2 desde entorno CICS

```text
Revisar transacciones en duda antes de forzar acciones.
Documentar identificador, usuario, terminal y hora.
```

## HSM: comprobaciones generales

```bash
ps -ef | grep -i hsm | grep -v grep
tail -f /var/log/<APP>/hsm.log
```

## Incidencias HSM

```bash
ps -ef | grep -i hsm | grep -v grep
grep -i "error\|fail\|timeout" /var/log/<APP>/hsm.log
```

## Contingencia transaccional

Documentar region, terminal, transaccion, hora de inicio, hora de fin y accion aplicada.

<!--
Fuentes consolidadas

- `cics.txt`
- `cics_db2_commands.txt`
- `cics_transaccions.txt`
- `alta_cics_user.txt`
- `consola_cics.txt`
- `console_msg.txt`
- `revisar_terminales_cics.txt`
- `error818.txt`
- `TPVs_fraude.txt`
- `hsm_logs.txt`
- `hsm_mpago.txt`
- `hsm_server.txt`
- `incidencia_hsm.txt`
- `HSM_MORA.txt`
- `intervencion_cicsp6.txt`
- `new 13.txt`
- `new 31.txt`
- `new 59.txt`
-->
