---
type: task
title: <título corto de la tarea>
description: <una línea: qué cambia y por qué>
tags: []
timestamp: AAAA-MM-DDTHH:MM:SSZ
x-mabbux:
  status: proposed          # proposed | accepted | retired · solo una persona pasa a accepted
  owner: "@albarracingarciap"
  provenance:
    author: agent           # agent | human
    agent: claude-code
---

# <título corto de la tarea>

> Carril: cambio. Se guarda como `work/AAAA-MM-DD-slug.md`. Si la tarea toca una ruta de
> `lanes.spec_required_paths`, no va aquí: abre una spec en `specs/`.

## Objetivo

<Qué se quiere conseguir, en una o dos frases.>

## Plan

1. <Paso.>

## Ficheros afectados

| Fichero | Acción | Motivo |
| --- | --- | --- |
| `<ruta>` | crear / modificar / borrar | <motivo> |

## Evidencia esperada

| Comprobación | Comando | Resultado esperado |
| --- | --- | --- |
| <qué demuestra> | `<comando>` | <código de salida 0, salida que contiene…> |

## Evidencia obtenida

Hasta que exista `done.sh`, se anota aquí y no en `.mabbux/state/evidencias.jsonl`.

| Comando | Código de salida | Resumen de la salida | Fecha |
| --- | --- | --- | --- |
| `<comando>` | <n> | <…> | AAAA-MM-DD |

## Conocimiento propuesto

<Documentos de `knowledge/` creados o modificados (con `status: proposed`), o «ninguno».>

## Notas y fricciones

<Supuestos, dudas o fricciones del proceso; las relevantes se copian a `knowledge/log.md`.>
