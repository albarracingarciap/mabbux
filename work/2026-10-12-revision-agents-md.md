---
type: task
title: Revisión de AGENTS.md antes del primer commit
description: Añade las secciones Git y Comandos, aclara el ciclo de vida de work/ y el significado de accepted, y registra la fricción de la lista de rutas duplicada
tags: [harness, agents-md]
timestamp: 2026-10-12T12:00:00Z
x-mabbux:
  status: accepted
  owner: "@albarracingarciap"
  provenance:
    author: agent
    agent: claude-code
---

# Revisión de AGENTS.md antes del primer commit

> Carril: cambio. Cambiar `AGENTS.md` nunca es trivial. El plan lo dio la persona
> responsable en el chat y se considera aprobado.

## Objetivo

Completar `AGENTS.md` con las reglas de git, el ciclo de vida de los documentos de tarea y
el estado de los comandos, sin pasar de 70 líneas.

## Plan

1. Añadir la sección «Git»: una rama por tarea (`feat/…`, `fix/…`, `docs/…`), nunca `main`,
   ni commit ni push sin petición expresa, y Conventional Commits en inglés.
2. Explicar el ciclo de `work/` y definir `accepted` como «revisado y aprobado por una
   persona» en `work/`, `specs/` y `knowledge/`.
3. Mantener la lista de rutas de spec y registrar en `knowledge/log.md` la fricción de
   tenerla duplicada.
4. Añadir la sección «Comandos»: aún no existen.

## Ficheros afectados

| Fichero | Acción | Motivo |
| --- | --- | --- |
| `AGENTS.md` | modificar | Puntos 1, 2 y 4; se compactan la tabla de rutas y algunos párrafos para no pasar de 70 líneas |
| `knowledge/log.md` | modificar | Fricción del punto 3 |
| `work/2026-10-12-revision-agents-md.md` | crear | Este documento |

## Evidencia esperada

| Comprobación | Comando | Resultado esperado |
| --- | --- | --- |
| Menos de 70 líneas | `wc -l < AGENTS.md` | < 70 |
| Existen las secciones nuevas | `grep -c -E '^## (Git\|Comandos)$' AGENTS.md` | 2 |
| `accepted` está definido | `grep -c 'revisado y aprobado por' AGENTS.md` | ≥ 1 |

## Evidencia obtenida

Hasta que exista `done.sh`, se anota aquí y no en `.mabbux/state/evidencias.jsonl`.

| Comando | Código de salida | Resumen de la salida | Fecha |
| --- | --- | --- | --- |
| `wc -l < AGENTS.md` | 0 | 68 | 2026-10-12 |
| `grep -c -E '^## (Git\|Comandos)$' AGENTS.md` | 0 | 2 | 2026-10-12 |
| `grep -c 'revisado y aprobado por' AGENTS.md` | 0 | 1 | 2026-10-12 |

## Conocimiento propuesto

Una entrada de fricción en `knowledge/log.md` (con `status: proposed`).

## Notas y fricciones

- El trabajo se ha hecho en `main`, aunque la nueva regla lo prohíbe. La sesión empezó
  antes de que existiera y no se ha hecho commit. Hay que crear una rama antes del primer
  commit (por ejemplo, `docs/harness-layer`).
- La primera sesión (la que creó la capa del harness) no tiene documento en `work/`.
