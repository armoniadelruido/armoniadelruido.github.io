---
title: Scripting util curl awk bash y parseos
date: 2026-07-02 01:40:00
categories: [Sistemas, Scripting]
tags: [sysadmin, bash, awk, curl, regex, scripting]
---

# Scripting util curl awk bash y parseos

Este post reune patrones utiles para automatizar tareas, probar endpoints HTTP, parsear logs y construir pequenos scripts shell.

## curl

```bash
curl -Ik https://app.example.local/
curl -sS https://app.example.local/api/health
curl -X POST https://app.example.local/api \
  -H 'Content-Type: application/json' \
  -d '{"clave":"valor"}'
```

## awk

```bash
awk '{print $1}' fichero.log
awk -F';' '{print $1,$3}' fichero.csv
awk '$9 ~ /^5/ {print $0}' access.log
```

## regex utiles

```bash
grep -E 'ERROR|WARN|FATAL' app.log
grep -Eo '([0-9]{1,3}\.){3}[0-9]{1,3}' app.log | sort -u
```

## parsear logs HTTP

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
awk '{print $9}' access.log | sort | uniq -c | sort -nr
```

## plantilla bash

```bash
#!/usr/bin/env bash
set -euo pipefail

main() {
  local fichero="${1:-}"
  if [[ -z "$fichero" ]]; then
    printf 'Uso: %s <fichero>\n' "$0" >&2
    exit 1
  fi

  grep -E 'ERROR|WARN' "$fichero"
}

main "$@"
```

<!--
Fuentes consolidadas

- `ansible.txt`
- `awk_bbdd.txt`
- `awakas.txt`
- `regex.txt`
- `VI_comands_regex.txt`
- `parseo_archivos.txt`
- `CURL.txt`
- `Python.txt`
- `POWERSHELL.txt`
- `COMPILAR_DE_BASH.txt`
- `script_curl_wsld.txt`
- `script_https.txt`
- `script_paths.txt`
- `script_sftp_3_equipos.txt`
- `script_2_cirro.txt`
-->
