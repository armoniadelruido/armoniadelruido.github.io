---
title: Plantilla segura para scripts bash de mantenimiento
date: 2026-07-06 00:40:00
categories: [Sistemas, Scripting]
tags: [bash, scripting, seguridad, logs, cron]
---

# Plantilla segura para scripts bash de mantenimiento

Este post resume patrones seguros para scripts de mantenimiento: logging, validaciones, `dry-run`, control de errores y limpieza de temporales.

## Plantilla base

```bash
#!/usr/bin/env bash
set -euo pipefail

LOG="/var/log/mantenimiento.log"
DRY_RUN="${DRY_RUN:-0}"

log() {
  printf '[%s] %s\n' "$(date --iso-8601=seconds)" "$*" | tee -a "$LOG"
}

need() {
  command -v "$1" >/dev/null 2>&1 || {
    log "Falta binario requerido: $1"
    exit 2
  }
}

run() {
  log "+ $*"
  if [[ "$DRY_RUN" != "1" ]]; then
    "$@"
  fi
}

main() {
  need rsync
  run rsync -avh /origen/ /destino/
}

main "$@"
```

## Temporales y limpieza

```bash
tmp="$(mktemp)"
trap 'rm -f "$tmp"' EXIT
```

## Leer ficheros linea a linea

```bash
while IFS= read -r linea; do
  printf 'Procesando: %s\n' "$linea"
done < fichero.txt
```

## Comprobar codigos de salida

```bash
if comando; then
  log "Comando correcto"
else
  log "Comando fallo"
  exit 1
fi
```

## Errores a evitar

- Variables sin comillas.
- Secretos escritos en el script.
- `rm` sin validar ruta.
- `kill -9` como primera opcion.
- Shebang incorrecto.
- Scripts sin `dry-run` para operaciones destructivas.

## Scripts relacionados

- `liberamemoria.sh`: ejemplo de mantenimiento simple con privilegios de root.
- `depura_ficheros.sh`: ejemplo de limpieza programada de ficheros antiguos.

<!--
Fuentes consolidadas

- `/tools/scripts/prueba_secuencia.sh`
- `/tools/scripts/prueba_secuencia_v2.sh`
- `/tools/scripts/parse_fichero/prova.sh`
- `/tools/scripts/parse_fichero/rr.sh`
- `/tools/scripts/scp_red.bash`
- `/tools/scripts/dni.py`
- `/tools/scripts/clean-snap.sh`
- `/tools/scripts/fix_modulo_propietario_fedora_warning.sh`
- `/tools/scripts/liberamemoria.sh`
- `/tools/scripts/scripts/libera_swap.sh`
- `/tools/scripts/scripts/pasa_pass.sh`
- `/tools/scripts/actualiza_firefox.sh`
- `/tools/scripts/mata_firefox.sh`
-->
