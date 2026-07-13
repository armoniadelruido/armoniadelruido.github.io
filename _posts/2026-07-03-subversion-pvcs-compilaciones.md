---
title: Subversion PVCS y compilaciones
date: 2026-07-03 01:00:00
categories: [Sistemas, Desarrollo]
tags: [sysadmin, svn, subversion, pvcs, cobol, compilacion]
---

# Subversion PVCS y compilaciones

Este post ayuda a trabajar con repositorios legacy, revisar cambios en Subversion/PVCS y documentar compilaciones o entregas tecnicas.

Antes de entregar, revisa `svn diff`, asocia el cambio a un ticket si aplica, compila en limpio y valida el binario generado. Crea tags solo cuando la version final este confirmada; si se etiqueta demasiado pronto, luego cuesta saber que fue realmente a produccion.

## Subversion

```bash
svn checkout https://svn.example.local/repos/<PROYECTO> /ruta/ejemplo/<PROYECTO>
svn status
svn update
svn diff
svn log --limit 10
svn commit -m "descripcion del cambio"
```

## Ramas y tags

```bash
svn copy https://svn.example.local/repos/<PROYECTO>/trunk \
  https://svn.example.local/repos/<PROYECTO>/tags/<VERSION> \
  -m "tag <VERSION>"
```

## Compilacion legacy

```bash
cd /ruta/ejemplo/<PROYECTO>
make clean
make
```

## Trazabilidad

Registrar siempre version, ticket, ruta de codigo y binario generado para mantener trazabilidad de la entrega.

<!--
Fuentes consolidadas

- `subversion_things.txt`
- `pvcs.txt`
- `new 25.txt`
- `new 45.txt`
- `new 50.txt`
-->
