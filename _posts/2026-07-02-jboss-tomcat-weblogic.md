---
title: JBoss Tomcat y WebLogic
date: 2026-07-02 00:20:00
categories: [Sistemas, Middleware]
tags: [sysadmin, jboss, tomcat, weblogic, java, ssl, ciphers]
---

# JBoss Tomcat y WebLogic

Este post ayuda a revisar procesos Java, localizar logs, configurar SSL y realizar comprobaciones basicas en JBoss, Tomcat, WebLogic y WAS.

## Revisar procesos Java

```bash
ps -ef | grep -E 'java|jboss|tomcat|weblogic' | grep -v grep
jps -lv
```

## Logs habituales

```bash
tail -f /opt/<APP>/logs/server.log
tail -f /opt/<APP>/logs/catalina.out
tail -f /opt/<APP>/domains/<DOMINIO>/servers/<SERVIDOR>/logs/<SERVIDOR>.log
```

## Tomcat: keystore y conector SSL

```xml
<Connector port="8443"
           protocol="org.apache.coyote.http11.Http11NioProtocol"
           SSLEnabled="true"
           keystoreFile="/opt/tomcat/conf/app.jks"
           keystorePass="<SECRETO>"
           clientAuth="false"
           sslProtocol="TLS" />
```

## JBoss: realm seguro

```bash
/opt/jboss/bin/jboss-cli.sh --connect
/subsystem=elytron/key-store=httpsKS:add(path=app.jks,relative-to=jboss.server.config.dir,credential-reference={clear-text="<SECRETO>"},type=JKS)
/subsystem=elytron/key-manager=httpsKM:add(key-store=httpsKS,credential-reference={clear-text="<SECRETO>"})
/subsystem=elytron/server-ssl-context=httpsSSC:add(key-manager=httpsKM,protocols=["TLSv1.2","TLSv1.3"])
reload
```

## Ciphers

Priorizar TLS moderno y retirar protocolos antiguos:

```text
TLSv1.2
TLSv1.3
```

Evitar cuando sea posible:

```text
SSLv2
SSLv3
TLSv1.0
TLSv1.1
RC4
3DES
NULL
EXPORT
```

## WebLogic: WLST basico

```python
connect('<USUARIO>', '<SECRETO>', 't3://admin.example.local:7001')
domainRuntime()
serverRuntime()
disconnect()
exit()
```

## WebLogic datasource

```python
connect('<USUARIO>', '<SECRETO>', 't3://admin.example.local:7001')
edit()
startEdit()
cd('/JDBCSystemResources/<DATASOURCE>/JDBCResource/<DATASOURCE>')
disconnect()
exit()
```

## WAS

```bash
serverStatus.sh -all
stopServer.sh <SERVIDOR>
startServer.sh <SERVIDOR>
```

<!--
Fuentes consolidadas

- `tomcat 7_ciphers.txt`
- `KeystoreTomcat.txt`
- `Artic_Tomcat.txt`
- `ciphers_Jboss.txt`
- `jboss_secure_realm.txt`
- `aliasJBOSS.txt`
- `Intervencion_JBOSS.txt`
- `controles_weblogic.txt`
- `weblogic_wslt.txt`
- `error_log_jboss.txt`
- `was_assegurances.txt`
- `new 7.txt`
- `new 8.txt`
- `new 23.txt`
- `new 52.txt`
- `new 55.txt`
- `new 57.txt`
- `new 77.txt`
-->
