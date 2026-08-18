---
title: Plataforma de parcheo Red Hat por CVEs con NSH y repositorio local
date: 2026-07-11 00:10:00
categories: [Sistemas, Seguridad]
tags: [redhat, yum, cves, parches, nsh, truesight, repositorios]
---

# Plataforma de parcheo Red Hat por CVEs con NSH y repositorio local

Este procedimiento documenta una plataforma de automatizacion de parches Red Hat basada en repositorios internos, metadata de seguridad de YUM y ejecucion remota con NSH/TrueSight.

La idea era separar el trabajo en dos fases: primero medir el estado de seguridad del parque, generar informes por servidor y consolidar un resumen; despues, con esa informacion, ejecutar el parcheo selectivo de las maquinas afectadas.

## Uso en la infraestructura

Encaja en entornos con muchos servidores Red Hat donde no se quiere que cada maquina salga directamente a Internet para descargar paquetes.

| Rol | Pieza | Impacto si falla |
|---|---|---|
| Repositorio local | YUM interno | Los servidores no ven parches ni erratas nuevas |
| Metadata de seguridad | `updateinfo.xml` | No se pueden mapear CVEs/RHSA pendientes |
| Orquestacion remota | NSH/TrueSight | No se puede consultar ni parchear el parque de forma centralizada |
| Reporting | informes por host | No hay visibilidad de riesgo ni priorizacion |
| Parcheo controlado | severidad Critical/Important | Quedan vulnerabilidades criticas sin tratar |

No era un flujo basado en Ansible ni en Red Hat Satellite. La automatizacion se apoyaba en scripts propios, comandos `yum` y ejecucion remota con `nexec`.

## Arquitectura

```text
Repositorio Red Hat externo
        |
        v
Servidor repositorio local
        |-- reposync / createrepo
        |-- updateinfo.xml / modifyrepo
        |-- repos rhel-6, rhel-7, rhel-secs
        |
        v
Servidores Red Hat del parque
        ^
        |
NSH / TrueSight
        |-- RHSA-list-sec-DES.nsh
        |-- format_resumen.sh
        |-- PARCHEA-RHSA-DES.nsh
        v
Informes por host + informe.csv + resumen
```

## Componentes

### Repositorio local

El servidor de repositorio descargaba paquetes y metadata desde Red Hat y los exponia en una URL interna. Los clientes Red Hat apuntaban a ese origen local.

```ini
[rhel-secs]
name=RHEL security local
baseurl=https://repos.example.local/rhel-secs/
gpgcheck=0
sslverify=0
enabled=1
```

La parte importante no era solo tener paquetes RPM, sino conservar la metadata de seguridad que permite a YUM responder preguntas sobre CVEs, RHSA y severidades.

```bash
yum -y install yum-utils createrepo yum-plugin-security

cp /var/cache/yum/x86_64/7Server/rhel-7-server-rpms/*updateinfo.xml.gz /ruta/repos/rhel-secs/repodata/
gzip -d /ruta/repos/rhel-secs/repodata/*-updateinfo.xml.gz
mv /ruta/repos/rhel-secs/repodata/*-updateinfo.xml /ruta/repos/rhel-secs/repodata/updateinfo.xml

cp /var/cache/yum/x86_64/7Server/rhel-7-server-rpms/repomd.xml /ruta/repos/rhel-secs/repodata/
modifyrepo /ruta/repos/rhel-secs/repodata/updateinfo.xml /ruta/repos/rhel-secs/repodata/
createrepo /ruta/repos/rhel-secs/
```

Con esa metadata disponible, cada host podia consultar seguridad sin salir a Internet.

### Consultas de seguridad

Los comandos base eran los de `yum-plugin-security`.

```bash
yum updateinfo list available
yum updateinfo list security all
yum updateinfo list cves
yum --security check-update
yum updateinfo summary
```

Para aplicar cambios de forma dirigida:

```bash
yum update --security
yum update-minimal --security
yum update --cve CVE-2008-0947
yum update --advisory=RHSA-2019:0163
```

La plataforma se apoyaba en esa salida para saber que servidores estaban afectados y que nivel de urgencia tenia cada parche.

### NSH y ejecucion remota

NSH/TrueSight aportaba la capa de orquestacion. Desde un punto central se recorria una lista de hosts y se ejecutaban comandos remotos con `nexec`.

Esqueleto original del enfoque:

```bash
#!/usr/bin/nsh
for host in `cat /ruta/origen/host_list`
do
  echo $host
  #nexec $host uname -a
  nexec $host yum updateinfo list cves > /ruta/destino/reports/$host
done
```

## Fase 1: analisis e informe

El primer script documentado era `RHSA-list-sec-DES.nsh`. Su objetivo era generar una foto del estado de parches de seguridad por host.

Flujo operativo:

1. Lee la lista de servidores desde un fichero de hosts.
2. Recorre cada host con un bucle.
3. Consulta la release del sistema remoto.
4. Limpia cache de YUM en el host.
5. Ejecuta `yum updateinfo list cves` o consultas equivalentes de seguridad.
6. Guarda la salida en un fichero con el nombre del host.
7. Ejecuta un script auxiliar remoto en `/tmp` para saber si tras el update haria falta reiniciar.
8. Genera un `informe.csv` consolidado.
9. Copia el informe a una maquina de presentacion.
10. Lanza `format_resumen.sh` para preparar un resumen legible.

Version revisada del patron de analisis:

```bash
#!/usr/bin/nsh

HOST_LIST="/ruta/origen/host_list_des"
REPORT_DIR="/ruta/destino/reports"
INFORME="/ruta/destino/informe.csv"

mkdir -p "$REPORT_DIR"
printf 'host;release;cves_pendientes;requiere_reboot\n' > "$INFORME"

for host in `cat "$HOST_LIST"`
do
  echo "Revisando $host"

  RELEASE=`nexec $host cat /etc/redhat-release 2>/dev/null`
  nexec $host yum clean all >/dev/null 2>&1
  nexec $host yum updateinfo list cves > "$REPORT_DIR/$host"

  CVES=`grep -c 'CVE-' "$REPORT_DIR/$host" 2>/dev/null`
  REBOOT=`nexec $host /tmp/requiere_reboot.sh 2>/dev/null`

  printf '%s;%s;%s;%s\n' "$host" "$RELEASE" "$CVES" "$REBOOT" >> "$INFORME"
done

/ruta/origen/format_resumen.sh "$REPORT_DIR" "$INFORME"
```

## Fase 2: parcheo por severidad

El segundo script documentado era `PARCHEA-RHSA-DES.nsh`. Partia de los ficheros generados en la fase de analisis.

No parcheaba a ciegas todo lo disponible. La idea era filtrar severidades y actuar solo sobre actualizaciones de seguridad relevantes, especialmente:

- `Critical`
- `Important`

Flujo operativo:

1. Lee el directorio de informes por host.
2. Por cada fichero, obtiene el nombre del servidor.
3. Busca parches instalables con severidad `Critical` o `Important`.
4. Ejecuta el update en el host correspondiente.
5. Registra la salida en un log.
6. Revisa si queda pendiente reinicio.

Version revisada del patron de parcheo:

```bash
#!/usr/bin/nsh

REPORT_DIR="/ruta/destino/reports"
LOG_DIR="/ruta/destino/logs"
mkdir -p "$LOG_DIR"

for report in "$REPORT_DIR"/*
do
  host=`basename "$report"`
  log="$LOG_DIR/parcheo_${host}.log"

  if grep -E 'Critical|Important' "$report" >/dev/null 2>&1
  then
    echo "Parcheando $host" | tee -a "$log"
    nexec $host yum -y update-minimal --security >> "$log" 2>&1
    nexec $host needs-restarting -r >> "$log" 2>&1
  else
    echo "Sin parches Critical/Important para $host" >> "$log"
  fi
done
```

## Reporting

La parte de reporting tenia dos niveles:

- fichero individual por host con la salida de `yum updateinfo`;
- informe consolidado en CSV;
- resumen formateado para una maquina de presentacion.

Esto permitia separar decision y ejecucion. Primero se podia revisar el parque y priorizar. Despues se podia lanzar el parcheo con criterios claros.

Ejemplo de CSV esperado:

```csv
host;release;cves_pendientes;requiere_reboot
host1;Red Hat Enterprise Linux Server release 7.x;12;no
host2;Red Hat Enterprise Linux Server release 7.x;3;yes
host3;Red Hat Enterprise Linux Server release 6.x;0;no
```

## Validaciones necesarias

Antes de parchear, el circuito debia comprobar:

- que el host responde a NSH;
- que el repositorio local esta configurado y accesible;
- que `yum updateinfo` devuelve metadata de seguridad;
- que hay espacio suficiente en `/var`, `/boot` y `/`;
- que el host no tiene transacciones YUM bloqueadas;
- que el informe generado no esta vacio;
- que existe ventana de mantenimiento para hosts con reinicio pendiente.

Despues de parchear:

- guardar log por host;
- volver a ejecutar `yum updateinfo list cves`;
- revisar `needs-restarting -r`;
- registrar si queda reinicio pendiente;
- comparar CVEs antes/despues.

## Riesgos

| Riesgo | Control |
|---|---|
| metadata `updateinfo.xml` desactualizada | refrescar repos antes de generar informes |
| salida de YUM vacia por repo mal configurado | validar `yum repolist` y `yum updateinfo summary` |
| parcheo masivo accidental | filtrar severidad y exigir lista de hosts controlada |
| falta de espacio | precheck de `/`, `/var` y `/boot` |
| reinicios pendientes no gestionados | registrar `needs-restarting -r` |
| informes inconsistentes | historico por fecha y fichero por host |


## Resumen

La plataforma combinaba repositorios Red Hat internos, consultas de seguridad de YUM y NSH/TrueSight para automatizar el ciclo de parches.

Automatizaba claramente:

- analisis de CVEs/RHSA pendientes;
- reporting por host;
- consolidacion en CSV/resumen;
- parcheo selectivo por severidad.

La descarga y publicacion de paquetes quedaba cubierta por el repositorio local. La validacion existia de forma parcial, centrada en salida de YUM, logs y deteccion de reinicio pendiente.

<!--
Fuentes consolidadas

- /home/alexaid/Documentos/wikis/Notepades/Circuito_actualizaciones.txt
- /home/alexaid/Documentos/wikis/Notepades/repositorios_local.txt
- /home/alexaid/Documentos/wikis/Notepades/Truesight.txt
- /home/alexaid/Documentos/Notepades_REVISION/notepades/updates_redhat.txt
- /media/site-web/armoniadelruido.github.io/_posts/2025-01-16-repo-local-redhat.md
-->
