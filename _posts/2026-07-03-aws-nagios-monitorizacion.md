---
title: AWS y monitorizacion Nagios
date: 2026-07-03 00:50:00
categories: [Sistemas, AWS]
tags: [sysadmin, aws, nagios, monitoring, cloud]
---

# AWS y monitorizacion Nagios

Este post cubre configuracion basica de AWS CLI, consultas de inventario y checks Nagios para servicios cloud o endpoints monitorizados.

## AWS CLI

```bash
aws configure --profile <PERFIL>
aws sts get-caller-identity --profile <PERFIL>
aws ec2 describe-instances --region <REGION> --profile <PERFIL>
```

## Inventario rapido

```bash
aws ec2 describe-instances \
  --region <REGION> \
  --profile <PERFIL> \
  --query 'Reservations[].Instances[].{id:InstanceId,state:State.Name,type:InstanceType}'
```

## Nagios: check TCP

```bash
/usr/lib/nagios/plugins/check_tcp -H <HOSTNAME> -p <PUERTO>
```

## Nagios: check HTTP

```bash
/usr/lib/nagios/plugins/check_http -H <HOSTNAME> -S -u /health
```

<!--
Fuentes consolidadas

- `aws_config.txt`
- `nagios_aws.txt`
- `inventario_nagios.txt`
- `new 2.txt`
-->
